# Errata: original CIS 546 report (Apr 2023)

1. **FordPass is listed as using certificate pinning, but it doesn't.**
   - The table shows "Yes. Certificate: src=system", which is only a system trust anchor.
   - Only DODGE has a `<pin-set>`. Results item 6 is corrected to "DODGE only".

2. **Debug-overrides were treated as production risks.**
   - `<debug-overrides>` only takes effect in debuggable builds.
   - Several apps listed under "allowing user-installed certificates" (results item 8) trust user CAs *only* inside debug-overrides. myCadillac and MyCar Controls are examples; their NSC files are shown in the report.
   - Of the NSC files shown, only Mercedes me connect trusts `user` CAs in a production (domain-config) block.
   - The remaining GM apps and Ferrari should be re-checked against their NSC files before a claim is made.

3. **"Did not implement NSC" was called a sign of lenience.**
   - A missing NSC file falls back to platform defaults, which are secure for apps targeting API 28 or later.
   - The report didn't record `targetSdkVersion`, so this conclusion is unsupported.

4. **The "Target android version" row is mislabeled.**
   - Values like "Android 8+" are the Play Store minimum supported version (minSdk), not the target SDK.
   - The row is relabeled accordingly.

5. **FordPass's user-certificate entry contradicts itself.**
   - The table says "reactivating trust for user installed CAs", but its CA configuration lists only `system`, and it is absent from the text list in item 8.
   - Re-check its NSC file.

6. **CarFax is listed under "custom CA configurations"** (item 7), but its table row shows only a debug-overrides `user` anchor.

7. **Cleartext exceptions weren't distinguished by risk.**
   - Tesla's exception is a single local-network IP (192.168.92.1).
   - myCadillac's exceptions are localhost, Android emulator hosts (10.0.2.2, 10.0.3.2), and three 198.208.3.x addresses.
   - These are far lower risk than app-wide cleartext and are now categorized separately.

8. **Other corrections.**
   - DODGE is missing its app number in item 7.
   - "Xth generation i7" in the test environment is a placeholder.
   - The method mentions javadecompilers.com, but the tables show only apktool.
   - APKs came from APKPure, a third-party mirror, and their integrity wasn't verified against the Play Store. This is now disclosed.

9. **The conclusion wasn't quantified.** "Most apps neglect the implementation" is replaced with counts: 6/20 app-wide cleartext, 1/20 production user-CA trust, 1/20 pinning.
