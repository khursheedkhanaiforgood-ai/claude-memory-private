---
name: feedback_eth0_uplink_ep1_trunk
description: AP connectivity issue traced to ETH0 uplink port mode set to Trunk in device-level EP1 setting (not a switch-side CLI problem)
type: feedback
originSessionId: b60a6aab-a768-4fff-b217-9bd45e71c887
---
Root cause of an AP troubleshooting session (2026-09-01, alongside 5320-16P-2MXT-2X-SwitchEngine work): the AP's ETH0 uplink was set to Trunk in the device-level EP1 setting. Correcting it resolved the issue.

**Why:** EP1's device-level port-mode setting for the AP's uplink port (ETH0) can cause connectivity problems that surface as confusing symptoms elsewhere (e.g. odd switch-side `show config` behavior while investigating), when the real cause is the AP's EP1 device-level uplink mode.

**How to apply:** When an AP has connectivity issues during onboarding and switch-side CLI checks come up clean, check the AP's device-level ETH0 uplink port mode in EP1 (Access vs Trunk) — don't assume the switch is the source of the problem just because that's where symptoms first appeared.
