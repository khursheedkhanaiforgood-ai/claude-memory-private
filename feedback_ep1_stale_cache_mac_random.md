---
name: EP1 Stale Cache + Ghost AP from Wrong Serial Claim
description: Client missing from show station on all known APs — actual root cause was ghost AP still active under wrong serial claim
type: feedback
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
`show station` on the AP is always authoritative. EP1 client association view is cached and can lag.

**Actual root cause (June 16 resolution):** Client was associated to a ghost device — AH-901340, briefly claimed in EP1 during the wrong-serial AP2 onboarding attempt. That device was still active in VIQ under the wrong serial, broadcasting the SSID, and handling client associations invisibly. Neither AP1 nor AP2 `show station` showed anything because the client was on a third device that wasn't supposed to exist.

**Fix:** Remove the wrong-serial device from EP1/VIQ. Client immediately appeared on the correct AP.

**Why:** Wrong serial claim during AP2 onboarding created a ghost AP (AH-901340, 172.16.0.147) that stayed active in VIQ even after the correct serial (AH-565780) was claimed.

**How to apply:**
- When a client is missing from `show station` on ALL APs: check EP1 device inventory for ghost devices (wrong serials, duplicate claims, devices that should have been removed).
- After any wrong-serial claim during onboarding, explicitly delete the ghost device from EP1/VIQ before proceeding.
- EP1 stale cache + macOS MAC randomization are still real causes — but exhaust ghost device check first when ALL APs show nothing.
- Full diagnostic chain: `show station` (all APs) → check EP1 device list for unexpected devices → `show fdb ports 3,5` → `networksetup -getmacaddress en0`
