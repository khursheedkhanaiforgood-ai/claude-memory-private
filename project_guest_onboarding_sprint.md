---
name: Guest Onboarding Sprint — May 12 2026
description: Week of May 12: Guest Onboarding with PPSK/XIQ in KhKLab. 802.1X/NPS deferred. PPSK guide v6 (Mike Rieben) is primary reference. VLAN 100 in use (user chose 100, not 80). Simulator live with step tracker + RFC links.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Sprint Goal
Deploy, test, break, and document guest onboarding options in KhKLab for client readiness.
Client requirement: guests separated from production, NOT on open SSIDs, PPSK preferred.

## Current Lab VLAN: 100
User created PPSK in XIQ with **VLAN 100** (not VLAN 80 as originally planned). SW1 needs VLAN 100 configured manually via EXOS CLI or via XIQ IQAgent (if SW1 is managed by XIQ).

## Why:** Client pressure to solve guest separation cleanly without 802.1X complexity.
## How to apply:** All suggestions orient around PPSK + VLAN isolation. 802.1X on hold.

## Deferred
- **802.1X / NPS deployment** → pushed to later sprint (NPS not ready, complexity deferred)

## Primary Reference Document
- File: /Users/khukhan/Downloads/Private-PreShared-Key-PPSK-Guide_v6 2 1.pdf
- Author: Mike Rieben, SA — Extreme Networks — v6 July 8 2022 — 95 pages
- This is an internal EN SA guide, NOT public. Do not push to GitHub.

## Lab Topology — Two Separate Internet Paths
```
PATH A — HAS INTERNET                PATH B — NO DIRECT INTERNET
AP1 → SW1 (EXOS 192.168.0.28)       AP2 → SW2/VOSS Port 1/3
    → QF-Modem (192.168.0.1)             → IPE Port 1/1 (SD-WAN)
    → Internet ✅                         → RDU/FishBowl DC (GRE/FE tunnel)
```
SW1↔SW2 physically linked via SW2 Port 1/10 (VLAN 4048/auto-sense).

## XIQ Instance Separation — CRITICAL
| Device | XIQ Instance | Internet | PPSK Cloud? |
|--------|-------------|---------|------------|
| AP1 | **MyLab_XIQ** | ✅ Yes | ✅ Full Cloud PPSK |
| AP2 (AP3000) | **MyLab_XIQ** | ✅ Yes | ✅ Full Cloud PPSK |
| SW2 (VOSS) | **FishBowl_XIQ** | ❌ No direct | N/A (switch only) |

## XIQ State (confirmed May 12 from screenshots)
- Network Policy: **AP_April21_KarlLab** (existing Karl Lab policy)
- SSID: `PPSK_Demo` — Private PSK — VLAN 100 ✅
- User Groups: `PPSK_Demo` (Cloud, 1 user) + `PPSK_Local` (Local, 2 users) ✅
- Default User Profile: `Guest_VLAN_100` (VLAN 100) ✅ — all PPSK users land on VLAN 100
- Assignment Rule: Contractor-3-days → Guest-VLAN-100 (existing Karl Lab rule)
- CWP: **SUSPECTED STILL ON** — first check tomorrow morning. If ON, AP blocks all non-HTTP traffic (ICMP/ping) until portal completed. This is the open blocker.
- **Firewall gap FIXED**: Rule order corrected May 12 — DENY RFC1918 (10.x, 172.16.x, 192.168.x) now rules 1-3, PERMIT ANY is rule 6. Deployed to AP1.
- Three SSIDs total: PPSK_Demo (VLAN 100), Guest_Wireless (VLAN 30, Enhanced Open), Corporate_Wireless (VLAN 20, WPA2/3)

## Two-Phase Deployment Plan (decided May 12)
### Phase 1 — CLI on SW1 (proven path, validate E2E)
1. `show iqagent` + `show lldp neighbors` → find AP1 port
2. Build VLAN 100 via EXOS CLI (SVI + DHCP):
```
create vlan Guest_100 tag 100
configure vlan Guest_100 ipaddress 10.100.0.1 255.255.255.0
enable ipforwarding vlan Guest_100
configure vlan Guest_100 add port <ap1-port> tagged
configure vlan Guest_100 dhcp-address-range 10.100.0.10-10.100.0.254
configure vlan Guest_100 dhcp-options default-gateway 10.100.0.1
configure vlan Guest_100 dhcp-options dns-server primary 8.8.8.8
configure vlan Guest_100 dhcp-options dns-server secondary 1.1.1.1
configure vlan Guest_100 dhcp-lease-timer 86400
enable dhcp ports <ap1-port> vlan Guest_100
save config
```
3. Add static route on QF-Modem: 10.100.0.0/255.255.255.0 → gateway 192.168.0.28
4. SSID already live on AP1 from Karl Lab policy — no new CAPWAP push needed
5. Create test user in XIQ → email PPSK → connect → verify 10.100.0.x → verify internet

### Phase 2 — XIQ Policy Push (production method)
1. Wipe manual SW1 config (delete vlan Guest_100)
2. XIQ → AP_April21_KarlLab → Step 3 (Switching/Routing) → Switch Template → add VLAN 100 + port tagged
3. Supplemental CLI in Step 3 → paste SVI + DHCP commands
4. Assign SW1 to policy → Deploy → XIQ pushes via IQAgent to SW1 + CAPWAP to AP1
5. Verify same E2E result — zero SSH to switch

## Routing — Home Lab
- Guest VLAN 100 (10.100.0.x) needs route back from QF-Modem
- Static route on QF-Modem (192.168.0.1): 10.100.0.0/255.255.255.0 → 192.168.0.28
- Without this: outbound works, return packets die at QF-Modem (no route to 10.100.0.x)
- Same issue hit with Corp_Guest VLAN in previous lab work

## Production Architecture Insight (May 12)
In production NO manual CLI to switches. XIQ handles via:
- AP config: CAPWAP (SSID, User Profile, VLAN tag, FW policy)
- Switch config: IQAgent + Step 3 Switch Template (VLAN, port membership)
- DHCP: Separate enterprise DHCP server (Infoblox, Windows DHCP) — not EXOS
- Routing: Perimeter firewall/router — not edge switch
Phase 4a wizard must call 4 APIs: XIQ AP + XIQ Switch + DHCP + Router

## PPSK Architecture — Key Derivation
- PMK = PBKDF2(HMAC-SHA1, PPSK, SSID_name, 4096, 256-bit) — RFC 2898/PKCS#5
- XIQ pre-computes PMK at provisioning, stores it. AP gets PMK via RadSec at runtime.
- PPSK never crosses the air. MIC in M2 is the only proof-of-possession OTA.

## PPSK Architecture — Runtime Flow
1. UE associates (open system, port LOCKED)
2. M1 ANonce → UE computes PMK+PTK locally
3. M2 SNonce+MIC → AP STUCK (no PMK stored in Cloud mode)
4. AP calls XIQ via RadSec (RFC 6614, TCP 2083) — Access-Request, User-Name=MAC
5. XIQ returns Access-Accept: PMK (MS-MPPE-Recv-Key, RFC 2548) + VLAN (Attr 81, RFC 2868) + FW policy
6. AP verifies M2 MIC, sends M3 GTK, receives M4 ACK → port OPEN, VLAN tagged
7. DHCP from SW1 VLAN 100 pool → guest online

## XIQ Object Hierarchy
```
SSID → User Group (Cloud, 24h first_login, 10-char PPSK)
     → User Profile (VLAN 100 + Guest-Internet-Access-Only FW)
     → Assignment Rule: UG → UP
     → User (per-person PPSK, email+SMS delivery)
```
**Key trap**: User Group must be inside SSID config (not standalone) for email delivery to work.

## Phase 1 Status — COMPLETE ✅ (resolved May 13)
- DHCP confirmed: all 3 VLANs (20/30/100) ✅
- RadSec auth: 4-Way Handshake + VLAN 100 tagged ✅
- Gateway reachable from all VLANs ✅
- **ping 8.8.8.8 from PPSK_Demo (VLAN 100): confirmed working ✅**
- **traceroute to google.com from MacBook: full route visible ✅**
- Guest VLAN 30 internet confirmed ✅
- Corporate VLAN 20 internet confirmed ✅

## Root Cause — AP1 Data Plane Incident (May 13)
- AP1 accumulated corrupted PTK/station table state from multiple policy changes + manual resets (May 12-13)
- Unicast wired→wireless: broken silently. Broadcasts (DHCP): still worked. Masked the problem for hours.
- Fix: XIQ-triggered reboot (Monitor → Devices → AP1 → Utilities → Reboot) — forced full CAPWAP policy push
- Diagnostic smoking gun: `arp -an` on MacBook = empty (gateway MAC never arrived via AP)

## Firewall Status (May 13 — partially resolved)
- Guest-Internet-Access-Only ACL re-attached to Guest_VLAN_100 ✅
- Rule order confirmed: PERMIT DHCP → PERMIT DNS → DENY RFC1918 ×3 → PERMIT Any ✅
- 192.168.0.28 blocked ✅ / 8.8.8.8 reachable ✅
- ⚠️ OPEN ISSUE*: DENY 10.0.0.0/255.0.0.0 NOT blocking 10.20.0.1 and 10.30.0.1
  - Mask confirmed /8 (255.0.0.0) in XIQ UI — should cover all 10.x but doesn't
  - Likely IQ Engine data plane bug or quirk with broad masks for intra-lab 10.x traffic
  - Fix: add explicit DENY /24 rules for each lab subnet (10.10, 10.20, 10.30, 10.40, 10.80)
  - Parked — will resolve in a dedicated pass

## Next Phase (from boss — May 13)
- **Task 1 — KhKLab**: Deploy CAPWAP configuration for PPSK_Demo SSID (in own lab/MyLab_XIQ)
- **Task 2 — Boss's VIQ**: Provision new AP in boss's lab/VIQ, deploy CAPWAP with PPSK

## Still Pending
- Add WPA2/WPA3 transition mode to PPSK_Demo SSID (macOS Ventura+ hides WPA2-only)
- PPSK Provisioning Model 2 (Open Guest_Register → CWP → email → PPSK_Demo)
- PPSK Provisioning Model 3 (Group PPSK / QR code)
- Phase 2: XIQ Policy Push for SW1 (move VLAN 100 from manual CLI to XIQ Switch Template)

## PPSK Provisioning Models — Sprint Order
| Phase | Model | Description | Status |
|-------|-------|-------------|--------|
| 1 | Single user | Receptionist/admin creates one user in XIQ, PPSK emailed/texted. Used for testing (user tested with own account). | ✅ DONE |
| 2 | Walk-in self-service | Open SSID (Guest_Register) → CWP self-registration → enter email/phone → XIQ creates user → PPSK sent → guest returns to CWP → enters PPSK → connected | 🔨 BUILDING NOW |
| 3 | Pre-registered QR code | Admin creates user in XIQ ahead of time → QR code generated → printed/emailed to guest → scan to auto-connect | User will do independently |
| 4 | Bulk CSV import | XIQ → Configure → Users & SSIDs → Import CSV → mass PPSK generation + email/SMS blast | User will do independently |

## CWP vs CAPWAP — Critical Distinction (confirmed May 13)
- **CAPWAP** = AP↔XIQ control protocol (UDP 5246/5247) — infrastructure, not guest-facing
- **CWP** = Captive Web Portal — guest-facing splash page
- Walk-in flow uses CWP on an Open SSID (Guest_Wireless), NOT CAPWAP
- "Deploy CAPWAP" = separate task = pushing clean AP policy from XIQ (infrastructure work)

## CAPWAP — RFC Reference
- **RFC 5415** — CAPWAP Protocol Specification (core spec, split-MAC, DTLS tunnel)
- **RFC 5416** — CAPWAP Binding for 802.11 (Wi-Fi specific extensions)
- **RFC 5417** — CAPWAP Access Controller DHCP Option 43 (AP discovery via DHCP)
- **RFC 6347** — DTLS (Datagram TLS — secures the CAPWAP control channel)
- Transport: UDP 5246 (control) + UDP 5247 (data)
- AP discovery sequence: DHCP Option 43 → DNS (redirect.aerohive.com) → manual
- Split-MAC: AP handles real-time 802.11 (beacons, ACK, crypto); XIQ handles policy/auth decisions

## CWP Self-Registration — Status (May 13, parked)
- Guest_Wireless (VLAN 30, Enhanced Open) = designated lobby SSID for walk-in flow
- Self-Registration toggle visible in XIQ SSID config ✅
- CWP pre-auth gives 192.168.0.3 (QF-Modem DHCP) — quarantine state, IP should change to 10.30.x after CWP accept — NOT verified yet
- Walled Garden needs XIQ domains: *.aerohive.com, *.extremecloudiq.com
- Self-registration fields (email/phone, User Group, delivery) live on SSID page not CWP template
- Parked — clarification needed from boss on self-registration User Group assignment to PPSK_Demo

## QR Code Notes (May 13)
- XIQ generates per-user QR code: WIFI:T:WPA2;S:PPSK_Demo;P:<PPSK>;;
- Location: Configure → Users & SSIDs → user → QR code icon
- Guest scans → auto-connects, no typing required

## macOS Client Issue (discovered May 12)
- macOS Ventura/Sonoma/Sequoia HIDES WPA2-only SSIDs from scan list
- PPSK_Demo (WPA2-only) invisible on both MacBooks; iPhone sees it fine
- Fix: add WPA2/WPA3 transition mode in XIQ SSID config
- Workaround: manually join via Wi-Fi → Other Network (but Join button also grayed out on macOS for WPA2-only)

## MAC Randomization Decision
- No MAC binding for guests (iOS/Android randomize per SSID)
- MAC binding ONLY for IoT/managed devices with fixed MACs
- PPSK validation is passphrase/PMK-based — MAC randomization is irrelevant

## Simulator State
- ppsk_simulator.html: 3 scenes, 24 steps, step tracker, clickable RFC links
- Step tracker: all steps visible, click to jump, ✓/▶/· status icons
- Phase 2/3: Node.js + Railway (tomorrow)
- Phase 4/4a: NVIDIA + XIQ Wizard (2-week sprint, manager approval)

## RFC Index
| RFC | Standard | Role in PPSK |
|-----|----------|-------------|
| RFC 2898 | PBKDF2 | PMK = PBKDF2(PPSK, SSID, 4096, 256) |
| IEEE 802.11i | RSN | 4-Way Handshake, PTK/GTK hierarchy |
| IEEE 802.11-2020 | Wi-Fi | Association, Beacon, RSN IE, CCMP |
| RFC 6614 | RadSec | AP→XIQ per-auth, TCP 2083 |
| RFC 2865 | RADIUS | Access-Request/Accept payload |
| RFC 2548 | MS-MPPE | PMK delivery in Access-Accept |
| RFC 2868 | Tunnel attrs | VLAN (Attr 81) in Access-Accept |
| RFC 5415 | CAPWAP | AP↔XIQ config push, UDP 5246 |
| RFC 2131 | DHCP | IP assignment after port OPEN |
| IEEE 802.1Q | VLAN | Guest VLAN tag on SW1 |
| IEEE 802.1X | Port-NAC | Controlled port model |

## Socratic Protocol (agreed May 12)
1. User states logic in own words
2. User writes pseudo-code / algorithm
3. Together formulate XIQ sequence of operations
4. CLI only if XIQ cannot push it
