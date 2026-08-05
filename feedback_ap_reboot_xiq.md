---
name: AP data plane fix — reboot from XIQ, not power cycle
description: After multiple policy changes and manual resets in XIQ, AP1 data plane can corrupt silently. Only a XIQ-triggered reboot pushes a clean policy and fixes it.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
After multiple policy changes (firewall add/remove, QoS add/remove) and manual AP resets, the AP (IQ Engine OS) can end up in a state where it stops delivering **unicast frames from the wired side to wireless clients**. Broadcasts still work (DHCP, gratuitous ARPs), which masks the problem.

**Why:** The AP's data plane state (VLAN-to-client mapping, encryption key setup, policy enforcement) accumulates partial/conflicting config across resets. A power cycle alone doesn't trigger a fresh XIQ policy push.

**How to apply:**
- A XIQ-triggered reboot (Monitor → Devices → AP → Utilities → Reboot) forces a clean CAPWAP reconnect and full policy re-push from XIQ.
- Use this FIRST when clients have DHCP but no internet after policy changes.
- Do NOT rely on power cycling the AP — it comes back up with stale state unless XIQ pushes fresh policy.
- After the XIQ reboot, wait 3-4 minutes before testing.
