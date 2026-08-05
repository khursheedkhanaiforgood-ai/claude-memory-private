---
name: EXOS Factory Reset Command
description: Correct EXOS SwitchEngine factory reset command vs wrong Fabric Engine command
type: feedback
originSessionId: 58be75a3-06c0-409a-950d-692d2c21abc7
---
Use `unconfigure switch all` for EXOS factory reset — NOT `delete /intflash/config.cfg`.

**Why:** `delete /intflash/config.cfg` is Fabric Engine (VOSS) syntax. On EXOS SwitchEngine it gives `%% Invalid input detected`. The correct EXOS command is `unconfigure switch all` which prompts for confirmation then wipes all config and reboots.

**How to apply:** Any time factory reset is needed on a 5320 running EXOS SwitchEngine, always use `unconfigure switch all`. Also always delete the switch from XIQ portal FIRST — otherwise XIQ recognises the serial number on ZTP+ boot and re-pushes the existing policy, undoing the wipe.
