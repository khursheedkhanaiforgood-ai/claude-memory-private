---
name: May 21 EP1 Zero-CLI Deploy Sprint
description: Stage 2 lab deploy from EP1 only. SW1 factory reset + AP1 wiped. Captures CLI output, gaps discovered, and EP1 workflow steps for EOD HTML.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Session Goal
Stage 2: deploy KhKLab two-SSID lab (Employee PPSK VLAN5 + Guest CWP VLAN10) entirely from EP1 UI.
SW1 = EXOS 5320-16P-2MXT-2X. AP1 = AP3000.

## Gaps Discovered (logged in xiq-ep1-transition.html)
- **G-01 (Bin #1):** EXOS switch factory reset NOT available from EP1. Requires CLI: `unconfigure switch all`. AP factory reset IS available: EP1 → Devices → AP → Actions → Factory Reset.

## AP1 Sequence (corrected)
Wrong: Delete from EP1 → physical pin reset
Correct: EP1 → Devices → AP1 → Actions → Factory Reset (single step, wipe + release)
AP1 claimed in EP1, factory reset pending (waiting for connection).

## SW1 Boot Output — Post Factory Reset

SW1 completed initial provisioning script on first boot. Full output for EOD HTML:

```
Do you want to see the list of CLI commands executed by this provisioning script? [y/N/q]: Yes
    disable telnet
    enable snmp access snmp-v1v2c
    configure snmpv3 add community "basic_ro" name dhaka2027 store-encrypted user v1v2c_ro 
    configure snmpv3 add community "basic_rw" name sylhet2027 store-encrypted user v1v2c_rw
    enable snmp access snmpv3
    configure snmpv3 add user babla authentication md5 privacy aes 128
    configure snmpv3 add group admin user babla sec-model usm
    configure failsafe-account
    configure failsafe-account permit telnet vr mgmt
    configure failsafe-account permit ssh vr mgmt

Do you want to see some basic CLI commands before entering the CLI? [y/N/q]: Yes
    Operation               Monitoring        Configuration
    ------------------      ------------      -----------------------------
    save configuration      show vlan         configure vlan add ports
    download image          show ports        configure ports
    ping                    show sharing      enable sharing
    reboot                  show log          configure stacking easy-setup

5320-16P-2MXT-2X-SwitchEngine.1 # save configuration
The configuration file primary.cfg already exists.
Do you want to save configuration to primary.cfg and overwrite it? (y/N) Yes
Saving configuration primary.cfg on master...... done!
5320-16P-2MXT-2X-SwitchEngine.2 #
```

**Key facts from provisioning script:**
- Telnet disabled automatically
- SNMP v1/v2c communities: basic_ro (dhaka2027), basic_rw (sylhet2027)
- SNMPv3 user: babla (MD5 auth, AES-128 privacy), group: admin
- Failsafe account: SSH + telnet on VR mgmt
- Config saved to primary.cfg
- SW1 prompt confirmed: `5320-16P-2MXT-2X-SwitchEngine.2 #`

## AP1 Connection — Confirmed

```
HiveManager Primary:    oh-cws-2.extremecloudiq.com
HiveManager Backup:     oh-cwm.extremecloudiq.com
HiveManager connection: Connected securely to Aerohive

CAPWAP client IP:        192.168.0.12
CAPWAP server IP:        3.145.235.80
VHM:                     VHM-UOZZMDJP
DTLS session:            Connected
Keepalives lost/sent:    0/36
Uptime:                  0w 0d 0h 5m 59s

IQ Engine:  10.8r6 build-2687fe7 (Feb 04 2026)
Platform:   AP3000-WW
```

**Key facts:**
- AP1 is fully connected to EP1 — should appear in Devices now
- VHM-UOZZMDJP = the org identifier (user's EP1 org)
- 192.168.0.12 = AP1's DHCP address on Default VLAN
- IQ Engine 10.8r6 = latest firmware as of Feb 2026

## SW1 Final Verified State — EP1 Push Complete ✓

Policy name: `SW1_EP1_VLAN5-10_Employee-Guest`

**show vlan (verified):**
```
Default         1    192.168.0.28/24   1/20 ports   VR-Default
VLAN5_Employee  5    10.5.0.1/24       f  0/1 ports  VR-Default
VLAN10_Guest    10   10.10.0.1/24      f  0/1 ports  VR-Default
f = IP Forwarding Enabled (pushed by EP1 natively)
```

**show dhcp-server (verified):**
```
VLAN5_Employee:
  DHCP Range:    10.5.0.10 → 10.5.0.254
  Gateway:       10.5.0.1
  DNS:           8.8.8.8 / 1.1.1.1
  Ports Enabled: 3 ✓

VLAN10_Guest:
  DHCP Range:    10.10.0.10 → 10.10.0.254
  Gateway:       10.10.0.1
  DNS:           8.8.8.8 / 1.1.1.1
  Ports Enabled: 3 ✓
```

**What EP1 pushed natively (no CLI needed):**
- VLAN creation (VLAN5_Employee tag 5, VLAN10_Guest tag 10)
- SVI IP addresses
- IP forwarding (flag f) — surprised us, XIQ did NOT do this
- Port 3 tagged to both VLANs
- DHCP address ranges + options (via Supplemental CLI)
- enable dhcp ports 3 (via Supplemental CLI, first-push only)

**Underscores preserved:** VLAN5_Employee / VLAN10_Guest intact — EP1 fixed the XIQ underscore-drop bug.

## Next Steps (in progress)
- [x] SW1 appears in EP1 Devices as unclaimed
- [x] Claim SW1 in EP1
- [x] AP1 factory reset from EP1
- [x] Build Switch Template in EP1 — SW1_EP1_VLAN5-10_Employee-Guest
- [x] Supplemental CLI pushed (DHCP pools + enable dhcp ports)
- [x] SW1 fully verified — all VLANs, IPs, DHCP confirmed
- [x] Gap G-03: EP1 disabled Port 3 on push — fixed with enable port 3 + save config
      **Critical lesson:** Always check Admin state on all ports after EP1 switch policy push.
      In EP1 port template: explicitly set Admin Enabled on AP-facing + uplink ports.
- [ ] Fix Port 3 in EP1 switch policy (set Admin Enabled) before next re-push
- [ ] Build AP1 Network Policy (Employee PPSK VLAN5_Employee + Guest CWP VLAN10_Guest)
- [ ] Push AP policy → validate E2E client test
</content>
