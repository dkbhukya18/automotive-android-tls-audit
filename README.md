# TLS Configuration Audit of 20 Automotive Android Apps

Course project for CIS 546, University of Michigan-Dearborn, Winter 2023.
Team: Dileep Kumar Bhukya, Deepa Akkala.

This is a **corrected** write-up of the original report. [ERRATA.md](ERRATA.md) lists every change.

## What the project does

Connected-car apps handle location, vehicle control and personal data. This project statically audited how 20 popular automotive Android apps configure TLS, using Android's **Network Security Configuration (NSC)**. NSC is an XML file that controls:

- Whether plain-HTTP (cleartext) traffic is allowed.
- Which certificate authorities (CAs) the app trusts, including CAs the user installs.
- Certificate pinning.
- Debug-only overrides.

**Method**
1. Download each APK from APKPure. This is a third-party mirror, and the downloads were not verified against the Google Play versions.
2. Decompile each APK with apktool.
3. Check `AndroidManifest.xml` for a `networkSecurityConfig` reference and for `android:usesCleartextTraffic`.
4. Analyze the referenced NSC XML file.

The analysis is static only. No traffic was intercepted, so the findings are configuration risks, not confirmed exploits.

## How to read NSC findings

Several of these points were misinterpreted in the original report.

- **No NSC file ≠ insecure.** Apps targeting API 24 or later don't trust user-installed CAs by default. Apps targeting API 28 or later also block cleartext by default. Without the app's `targetSdkVersion` (inside the APK), a missing NSC file tells you nothing on its own.
- **`<debug-overrides>` only applies to debuggable builds** (`android:debuggable="true"`). Store releases are not debuggable, so trusting user CAs *inside* debug-overrides is **not** a production weakness. Trusting user CAs in `<base-config>` or `<domain-config>` is.
- **Custom trust anchors (`@raw/...`) aren't pinning.** They add CAs to the trust store. Pinning requires a `<pin-set>`.
- **Cleartext for local addresses is often intentional.** Examples are an emulator host (10.0.2.2), localhost, or a vehicle's local Wi-Fi IP. That is lower risk than enabling cleartext for all domains.

## Findings

| Finding | Apps | Risk |
|---|---|---|
| Cleartext allowed for **all** domains (`base-config cleartextTrafficPermitted="true"`) | MyBMW, MyGMC, MyBuick, MyFerrari, MyCar Controls, Chrysler | **High:** any HTTP request leaves the device unencrypted |
| Cleartext allowed globally, blocked only for listed API domains | MyHyundai (Bluelink) | Medium: the core API is protected, everything else isn't |
| Cleartext allowed only for specific hosts | Tesla (192.168.92.1, a local-network address), myCadillac (localhost, emulator and test IPs), MyChevrolet, My Honda Moto | Low to medium: mostly local or test hosts. Leftover test entries are still worth flagging. |
| User-installed CAs trusted in a **production** config | Mercedes me connect (`user` listed in domain-config trust anchors) | **High:** anyone who can get a CA installed on the device (malware, MDM, a coerced user) can intercept this app's TLS traffic |
| User-installed CAs trusted **only in debug-overrides** | myCadillac, MyCar Controls, Chrysler, and other GM-template apps | Not a production issue, though it shows that debug configuration ships in release packages |
| Certificate pinning (`<pin-set>`) | **DODGE only** | Positive control |
| No NSC file | MySubaru, KiaConnect, InfoCar, Lexus | Can't be judged without `targetSdkVersion` (see above) |

**Summary.** 6 of 20 apps permit cleartext traffic app-wide, 1 of 20 trusts user-installed CAs in production, and only 1 of 20 pins certificates. The pinning result is the strongest signal: for apps that control physical vehicles, pinning should be standard, and 19 of 20 apps don't use it.

## Limitations and next steps

- The results are a 2023 snapshot of the app versions listed in the original tables, and current versions may differ.
- `targetSdkVersion`, `usesCleartextTraffic`, and any certificate pinning done in code (for example OkHttp's `CertificatePinner`) were not checked. NSC is not the only place an app can configure TLS.
- **Next step:** dynamic validation. Install a user CA, proxy traffic through Burp Suite, and test whether the high-risk apps actually accept the interception. Frida can show whether any pinning happens in code.

## References

1. Android Developers. *Network security configuration.* https://developer.android.com/privacy-and-security/security-config
2. AppSec Labs. *Understanding the Android cleartextTrafficPermitted flag.* https://appsec-labs.com/portal/understanding-the-android-cleartexttrafficpermitted-flag/
3. OWASP Mobile Application Security Testing Guide (MASTG): network communication testing. https://mas.owasp.org/MASTG/
