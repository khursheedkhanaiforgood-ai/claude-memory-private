---
name: macOS hides WPA2-only SSIDs
description: macOS Ventura/Sonoma/Sequoia filters WPA2-only SSIDs from the scan list and grays out the Join button for manual entry. iOS is permissive.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---

macOS Ventura+ deliberately hides WPA2-only SSIDs and refuses to join them even via manual entry (Join button grayed out). iPhone shows and joins WPA2 SSIDs without restriction.

**Why:** Apple deprecated WPA2-only in Ventura and enforcement strengthened in Sonoma/Sequoia as part of their WPA3 push.

**How to apply:** Any new SSID in the lab must be configured as **WPA2/WPA3 Personal (transition mode)** in XIQ to be visible and joinable on MacBooks. Pure WPA2 SSIDs will only work on iPhone/older devices. Always set transition mode as the default for new SSIDs.

Fix in XIQ: Network Policies → [policy] → [SSID] → Security → WPA2/WPA3 Personal (transition mode).
