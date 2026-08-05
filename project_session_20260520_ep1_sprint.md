---
name: May 20 EP1 Sprint — Employee + Guest SSIDs, GuestEssentials, AirDefense
description: May 20 2026 full-day EP1 sprint. Factory-reset SW1+AP1. Two SSIDs (Employee PPSK VLAN5, Guest Open+CWP VLAN10). Stage 1 = UE for AP1 + CLI for SW1. Stage 2 = ALL UE. GuestEssentials + AirDefense integrated. For EOD HTML.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Session Context
- Date: May 20, 2026
- Scope: ALL EP1 (Extreme Platform One) — no XIQ Classic
- Devices: SW1 (EXOS, KhursheedsLab, 192.168.0.28) + AP1 (AP3000, KhursheedsLab, Port 3 of SW1, SN: HA012519Y-10623)
- SW1 factory-reset at start of session (`unconfigure switch all`)
- AP1 factory-reset at start of session
- Router: QF-Modem at 192.168.0.1 (default gateway for VLAN 1)

## Network Design

### VLAN Map
| VLAN | Name     | Subnet         | DHCP Source | Purpose                    |
|------|----------|----------------|-------------|----------------------------|
| 1    | Default  | 192.168.0.x    | QF-Modem    | AP mgmt / SW1 management   |
| 5    | Employee | 10.5.0.0/24    | SW1         | PPSK employee traffic      |
| 10   | Guest    | 10.10.0.0/24   | SW1         | Open + CWP guest traffic   |

### SSID Map
| SSID             | Security          | VLAN | Schedule         | Access model                           |
|------------------|-------------------|------|------------------|----------------------------------------|
| KhKLab_Employee  | WPA2/WPA3 + PPSK  | 5    | Always on        | Pre-issued individual key · 5GHz only  |
| KhKLab_Guest     | Enhanced Open (OWE) + CWP | 10 | M–F 10:00–14:00 | 2-step: connect → CWP · 5GHz + 2.4GHz · OWE Transition Mode |

### Physical Topology
- QF-Modem (192.168.0.1) → SW1 Port 1 (uplink, VLAN 1 native)
- SW1 Port 3 → AP1 (trunk: VLAN 1 untagged native + VLAN 5 tagged + VLAN 10 tagged)
- AP1 bridges VLAN 5 (employee) and VLAN 10 (guest) back to SW1

### QF-Modem Static Routes (required for internet breakout)
```
10.5.0.0/24  → 192.168.0.28   (SW1 = VLAN 5 gateway)
10.10.0.0/24 → 192.168.0.28   (SW1 = VLAN 10 gateway)
```
Path: QF-Modem Advanced → Routing → Static Routes

## Stage 1 — UE for AP1 · CLI for SW1

### Step 1 — SW1 Post-Factory-Reset CLI
Run once manually over console/SSH. NOT idempotent — do NOT put enable dhcp / enable ipforwarding in EP1 Supplemental CLI template.

```bash
# Post-reset one-time commands
enable ports all
enable ipforwarding vlan Default

# Create VLANs
create vlan Employee tag 5
create vlan Guest tag 10

# SVI IP addresses (SW1 = default gateway for each VLAN)
configure vlan Employee ipaddress 10.5.0.1 255.255.255.0
configure vlan Guest ipaddress 10.10.0.1 255.255.255.0

# Port 3 trunk toward AP1 (VLAN 1 already native/untagged from factory reset)
configure vlan Employee add ports 3 tagged
configure vlan Guest add ports 3 tagged

# DHCP server for Employee (VLAN 5)
configure vlan Employee dhcp-address-range 10.5.0.10 - 10.5.0.254
configure vlan Employee dhcp-options default-gateway 10.5.0.1
configure vlan Employee dhcp-options dns-server primary 8.8.8.8
configure vlan Employee dhcp-options dns-server secondary 1.1.1.1

# DHCP server for Guest (VLAN 10)
configure vlan Guest dhcp-address-range 10.10.0.10 - 10.10.0.254
configure vlan Guest dhcp-options default-gateway 10.10.0.1
configure vlan Guest dhcp-options dns-server primary 8.8.8.8
configure vlan Guest dhcp-options dns-server secondary 1.1.1.1

# Enable services (NOT idempotent — manual only)
enable ipforwarding vlan Employee
enable ipforwarding vlan Guest

# Enable DHCP server on port 3 for each VLAN (NOT enable dhcp vlan <name> — that is client mode)
enable dhcp ports 3 vlan Employee
enable dhcp ports 3 vlan Guest

save config
```

**Verify after running:**
```bash
show vlan
show dhcp-server
# Confirm: Ports DHCP Enabled: 3 on both Employee and Guest VLANs
```

**Lesson learned:** `enable dhcp vlan <name>` = DHCP client mode (throws error if static IP configured).
DHCP server requires: `enable dhcp ports <port> vlan <name>` — different command entirely.
`show dhcp-server` "Ports DHCP Enabled: No ports enabled" = server configured but not listening — fix with ports command.

### Step 2 — QF-Modem Static Routes
Add two static routes so guest/employee VLANs can reach internet through SW1.
- `10.5.0.0/24` → next-hop `192.168.0.28`
- `10.10.0.0/24` → next-hop `192.168.0.28`

### Step 3 — EP1: Confirm SW1 IQAgent Connected
EP1 → Manage → Devices → SW1 should appear as Connected after reset + reconnect (~2–5 min).
IQAgent heartbeat "proxy device-connector unknown POST /health-check/[serial]" every ~60s = normal, NOT an error.

### Step 4 — EP1: Claim AP1
EP1 → Manage → Devices → AP1 appears in Unmanaged/New after CAPWAP reconnect.
Claim it, assign to correct location.

### Step 5 — EP1: Build Network Policy KhKLab_Stage1
EP1 → Configure → Network Policies → Add

**Policy name: AP1_ViaEP1_VLAN5-10_PPSK_5**

**SSID 1: KhKLab_Employee**
- Authentication: PPSK
- User Group: "Employees" — Cloud storage (NOT Local — Local limits to 1K keys)
- Create 2–3 test users with individual unique passphrases
- VLAN: User Profile → VLAN 5
- WPA2/WPA3 transition mode (prevents macOS Ventura hiding SSID)
- No inter-station traffic (blocked — employees isolated from each other)
- No BYOD (no MAC binding)

**SSID 2: KhKLab_Guest**
- Authentication: Open
- CWP: enabled → self-registration (2-step: connect → CWP → register → access granted)
- Schedule: Mon–Fri 10:00–14:00 (SSID broadcast window)
- VLAN: VLAN 10
- Firewall: DENY 10.0.0.0/8 → DENY 172.16.0.0/12 → DENY 192.168.0.0/16 → PERMIT any (internet only, top-down first-match)
- Inter-station traffic: DISABLED (Additional Settings → Optional Settings → Traffic Filters → uncheck Enable Inter-station Traffic)
- Max devices per key: 1 (Employee SSID only)

### Step 6 — EP1: Assign Policy to AP1 and Push
EP1 → Manage → Devices → AP1 → Assign Network Policy → KhKLab_Stage1 → Deploy

### Step 7 — Verification
**Employee test:**
```
Connect to KhKLab_Employee with PPSK → confirm IP in 10.5.0.x range
SW1: show fdb ports 3        → MAC should appear on VLAN 5
SW1: show iparp vlan Employee → client IP visible
```
**Guest test:**
```
Connect to KhKLab_Guest → CWP page loads → complete 2-step reg → confirm IP in 10.10.0.x range
SW1: show fdb ports 3        → MAC should appear on VLAN 10
```

## Stage 2 — ALL UE (EP1 Only — deferred after Stage 1 verified)
1. Factory reset SW1 again (`unconfigure switch all`)
2. Factory reset AP1
3. Re-onboard both via EP1 only (no CLI)
4. Use EP1 Switch Template to configure:
   - Create VLAN 5 + VLAN 10
   - Port 3 trunk (tagged VLAN 5 + VLAN 10)
   - DHCP via Supplemental CLI template (dhcp-address-range commands only — NOT enable dhcp / enable ipforwarding)
5. Deploy same network policy KhKLab_Stage1 to both devices from EP1

## Guest Essentials (to integrate during session)
- EP1 module for guest lifecycle management
- Features: CWP portal, sponsor approval, time-limited access, SMS/email notifications, usage reports
- Covers the KhKLab_Guest 2-step flow: guest arrives → connects → CWP → registers → admin/sponsor approves → time-limited access
- Navigation: EP1 → Configure → Guest Management (or Guest Essentials module)
- Key config: expiry time, approval workflow (self-serve vs sponsor), branding
- To be documented with actual EP1 navigation path during session

## AirDefense (to integrate during session)
- Extreme WIDS/WIPS — Wireless Intrusion Detection/Prevention
- Detects: rogue APs, unauthorized clients, RF attacks, deauth floods, jamming
- APs act as sensors: monitor mode (dedicated sensor) or background scanning (serving + scanning)
- In EP1: integrated under security/RF management — actual nav path to be confirmed during session
- Key decisions: alert-only vs auto-contain; scan interval; rogue AP classification rules
- To be documented with actual EP1 navigation path during session

## EOD HTML Notes (accumulate during session)
- Format: NYT journal style (white, Libre Baskerville serif) — NOT dark theme
- File: session_summary_20260520.html
- Must be linked from all 3 index pages (index.html, index-nyt.html, index-harpers.html)
- Sections needed:
  1. Session objective + context (factory reset, all EP1)
  2. Network design diagram (VLAN table + topology)
  3. Stage 1 — complete CLI block for SW1 (verbatim, copy-pasteable)
  4. Stage 1 — EP1 steps with navigation paths (anyone can follow)
  5. Verification commands (show fdb, show iparp)
  6. Guest Essentials — what it is + what we configured
  7. AirDefense — what it is + what we configured
  8. Stage 2 — EP1-only re-deploy steps
  9. Lessons learned
  10. Tomorrow's sprint / pending items

## Key Constraints / Gotchas (carry forward)
- `enable dhcp` and `enable ipforwarding` are NOT idempotent — run manually, NEVER in EP1 Supplemental CLI template
- PPSK must be typed manually — never paste (invisible chars break it)
- macOS Ventura+ hides WPA2-only SSIDs — use WPA2/WPA3 transition mode
- `show fdb ports 3` is truth for VLAN assignment (ARP can be stale up to 20 min)
- IQAgent heartbeat every ~60s = normal keepalive, not an error
- EXOS factory reset = `unconfigure switch all` (NOT `delete /intflash/config.cfg` — that is VOSS)
- Port 3 already in VLAN 1 untagged from factory reset — only need to add tagged VLANs 5 + 10
