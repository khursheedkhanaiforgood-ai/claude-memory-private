---
name: SSH host key changes after AP factory reset
description: After factory resetting an AP, SSH throws "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED" because the device generates a new host key. Fix is one command.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
After any AP factory reset, SSH will refuse connection with "REMOTE HOST IDENTIFICATION HAS CHANGED" (offending key line in ~/.ssh/known_hosts).

**Fix — run once:**
```
ssh-keygen -R <ap-ip-address>
```
Then reconnect normally:
```
ssh admin@<ap-ip-address>
```
Accept the new fingerprint when prompted.

**Why:** Factory reset wipes the AP's SSH host key. macOS strict host key checking detects the mismatch and blocks connection as a security measure. `ssh-keygen -R` removes the stale entry from `~/.ssh/known_hosts`.

**How to apply:** Any time SSH to an AP fails with "REMOTE HOST IDENTIFICATION HAS CHANGED" after a factory reset or firmware upgrade — run `ssh-keygen -R <ip>` first, then reconnect.
</content>
