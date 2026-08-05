---
name: Supplemental CLI idempotency — enable dhcp and enable ipforwarding must never be in the template
description: Both enable dhcp and enable ipforwarding are non-idempotent on EXOS. They hang XIQ at 15% on re-run. Both are manual post-factory-reset steps only, never in the Supplemental CLI template.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
Never put `enable dhcp ports X vlan Y` or `enable ipforwarding vlan Default` in the permanent XIQ Supplemental CLI template for EXOS switches.

**Why:** Both commands are non-idempotent on EXOS SwitchEngine. When XIQ pushes the Supplemental CLI a second time (policy update, delta deploy, etc.), EXOS throws an error on these lines and XIQ hangs at ~15% completion. The policy push stalls and never completes.

**The commands:**
```
enable dhcp ports 3 vlan VLAN10       ← NOT idempotent — hangs on re-run
enable ipforwarding vlan Default       ← NOT idempotent — same issue
```

**How to apply:**

These belong in a **Post-factory-reset checklist** — run manually ONCE before the very first XIQ deploy, then never again:

```
# Run these manually over SSH or console BEFORE the first XIQ deploy
enable ports all
enable ipforwarding vlan Default
save config
```

After `save config`, the config survives reboots. XIQ never needs to set it again. Remove both lines from any Supplemental CLI template permanently.

**The Supplemental CLI template should ONLY contain idempotent config** — commands that can be run 100 times without error. Examples: `configure vlan <name> dhcp-address-range`, `configure vlan <name> dhcp-options default-gateway`, `configure vlan <name> dhcp-lease-timer`. These are safe because EXOS silently overwrites on re-run.

**Discovery:** May 15 2026. Khursheed tried adding `enable ipforwarding vlan Default` to Supplemental CLI after noticing it caused the same 15% hang as `enable dhcp ports 3 vlan VLAN10` (which had been caught earlier). Both commands fail the same way for the same reason: they error if the feature is already enabled.

**Also applies to:** Any command prefixed with `enable` that has a corresponding `disable` — always check if EXOS errors on re-enable before adding to Supplemental CLI.
