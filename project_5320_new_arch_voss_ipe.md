---
name: 5320 New Architecture — VOSS + IPE + RDC Workstream
description: New workstream Apr 30 2026. SW2 VOSS 9.3.2.0.GA installed. XIQ DNS issue hit — fresh ZTP+ start planned. Branch: feature/new-arch-voss-ipe.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Branch
`feature/new-arch-voss-ipe` — branched from `main` Apr 30 2026.
NEVER merges to main until Phase 6 E2E test PASS.
All existing EXOS/VLAN 40-70 work stays on feature/auto-deploy-agent — completely separate.

## Architecture Docs
- HTML: `/Users/khukhan/5320-onboarding-agent/docs/arch_voss_ipe_overview.html`
- EOD: `/Users/khukhan/5320-onboarding-agent/docs/session_summary_20260430.html`

## Two-Path Topology
```
PATH A (unchanged)          PATH B (new architecture)
─────────────────           ─────────────────────────────────────
AP1 (192.168.0.12)          AP2 (192.168.0.25) ← FA auto-detected at boot
    ↓                           ↓
SW1 (192.168.0.28)          SW2-VOSS (192.168.0.31)
EXOS/SwitchEngine               ↓ Port 1 transit (10.0.1.1/30)
    ↓                       IPE — Physical Extreme Networks Box
HomeModem (192.168.0.1)         ↓ LAN 10.0.1.2 / WAN 192.168.0.30
    ↓                       HomeModem (192.168.0.1)
Internet                        ↓
                            FE-Tunnel → RDC Raleigh
```

## IPE Role Boundary (critical — never confuse)
| SW2 (VOSS) | IPE (Physical Box) |
|------------|-------------------|
| IS-IS, Nickname 0.00.02, I-SIDs, Fabric Attach | FE-Tunnel (Fabric Extend) configured here |
| Transit Port 1 → physical cable to IPE | IAH (application hosting) runs on IPE |
| Static route → IPE as next-hop | IPsec encapsulation of I-SIDs |
| NO tunnel, NO IAH config | WAN → HomeModem → RDC |

## Physical cabling (when IPE arrives):
`SW2 Port 1` → `IPE LAN` → `IPE WAN` → `HomeModem` → Internet → `RDC Raleigh`

## Session Apr 30 — What Happened
- VOSS 9.3.2.0.GA installed via XIQ ✅
- SW2 booted, AP2 auto-detected FA on port 1/3 ✅
- rwa password changed (default rwa/rwa → new) ✅
- Pre-flight CLI: hostname, VLAN 1 IP 192.168.0.31, default route, DNS ✅
- Internet confirmed: ping 8.8.8.8 alive ✅
- XIQ connectivity FAILED: IQAgent DNS broken after manual static IP config ❌
  - Root cause: `ip name-server` doesn't feed IQAgent's /etc/resolv.conf
  - TLS cert fails with IP address (cert is for hostname)
- Decision: WIPE SW2, fresh ZTP+ start next session

## Fresh Start Plan (Next Session)
1. Delete SW2 from XIQ inventory first
2. VOSS reset: `delete /intflash/config.cfg` + `reload`
3. Connect Port 1 to HomeModem BEFORE boot (DHCP at boot → DNS auto-configured)
4. ZTP+ connects to XIQ automatically
5. Accept in XIQ Inbox as Fabric Engine device
6. Push IS-IS + SPB + Nickname 0.00.02 + VLANs from XIQ

## Current Switch State (May 1 2026 — post-VIQ onboard)
Hostname: KhKLab-SW-01
Running config saved: /Users/khukhan/sw2_running_config_20260501.txt

### Management
- CLIP (loopback) IP: 10.158.4.1/32 — this is the mgmt IP, NOT a regular interface
- SSH to 10.158.4.1 requires routing via VIQ fabric network (10.152.x.x)
- VIQ network: DNS 10.152.0.11, syslog 10.152.0.110, domain RDUFishbowl.local

### VIQ-pushed config (confirmed in running-config)
- VLAN 4048 "onboarding-vlan" type pvlan-mstprstp 0 secondary 4049
- vlan i-sid 4048 15999999
- All ports 1/1–1/20: auto-sense enable
- router isis: spbm 1 ip enable, sys-name KhKLab-SW-01, is-type l1, enabled
- auto-sense onboarding i-sid 15999999
- i-sid name 15999999 "Onboarding I-SID"
- SNMP: FB-SNMP user, snmpuser, host 10.152.0.110
- cfm spbm enable, auto-sense auto-mlt enable

### Nickname — CONFIRMED ✅ (May 1 2026 19:42 EDT)
`show isis spbm` output: NICK NAME = 0.00.02, B-VID 4051-4052, PRIMARY 4051, IP enable, ORIGIN dynamic.
Full sequence that worked:
```
configure terminal
no router isis enable        ← must disable first (runtime change not allowed)
router isis
spbm 1 nick-name 0.00.02    ← hyphenated, inside router isis context
exit
router isis enable
exit
save config
```
Verify with: `show isis spbm` — look for NICK NAME column.

### VLAN 4047 "SD-WAN" on Port 1/1
NOT in running config — dynamically assigned by auto-sense from IPE FA handshake.
Do NOT manually configure VLAN 100 on Port 1/1 (conflicts with auto-sense managed port).

### Ports
- Ports 1/1–1/20: all auto-sense enabled
- VLAN 4048: Ports 1/3 (AP2), 1/10 (SW1) — untagged, auto-sense managed

## Corp/Guest SSIDs — Confirmed Values (May 1 2026)
- Corp: VLAN 70 "Corp_New", I-SID 100070, Gateway 10.70.0.1/24, DHCP 10.70.0.100–200
- Guest: VLAN 80 "Guest_New", I-SID 100080, Gateway 10.80.0.1/24, DHCP 10.80.0.100–200
- Convention: I-SID = VLAN + 100,000
- AP2 on Port 1/3 — FA auto-provisions tagged VLANs (no manual port tagging)
- DHCP hosted locally on SW2 (island mode — SW1 Port 10 disconnected during test)

## Service-at-the-Edge CLI (READY TO RUN Monday — full confirmed block)

### Step 1: Lock Area Address
```
configure terminal
router isis
  manual-area 49.b0b1
exit
```

### Step 2: Service VLANs + I-SID Mapping
```
vlan create 70 name "Corp_New" type port-mstprstp 0
vlan create 80 name "Guest_New" type port-mstprstp 0
vlan i-sid 70 100070
vlan i-sid 80 100080
```

### Step 3: DHCP (confirmed VOSS syntax — NOT NotebookLLM pool syntax)
```
ip dhcp-server subnet 10.70.0.0/24
  pool 10.70.0.100 10.70.0.200
  router 10.70.0.1
  domain-name-servers 8.8.8.8
exit
ip dhcp-server subnet 10.80.0.0/24
  pool 10.80.0.100 10.80.0.200
  router 10.80.0.1
  domain-name-servers 8.8.8.8
exit
ip dhcp-server enable
```
VOSS DHCP keywords confirmed: `subnet` (not pool), `pool <start> <end>`, `router` (not default-router), `domain-name-servers` ✅

### Step 4: Fabric Attach on Port 1/3
```
interface gigabitEthernet 1/3
  auto-sense enable
  fa enable
  no shutdown
exit
save config
```
After this: push SSID policy from VIQ → AP2 sends FA TLV → port 1/3 auto-tagged.
Verify with: show fa assignment  (look for Port 1/3, Active, Dynamic, I-SIDs 100070/100080)

## IPE Transit Path (PENDING — need IPE lan1 IP)
VIQ put VLAN 4047 (SD-WAN) on Port 1/1. IPE wan2 = 192.168.0.20/24 (confirmed from dashboard).
IPE lan1 IP = UNKNOWN — must discover before setting default route.

### NotebookLLM Step 4 errors (do NOT use):
- VLAN 30 "IPE_Handoff" — unconfirmed, may conflict with VIQ auto-sense config
- 192.0.2.x — RFC 5737 documentation placeholder, not a real address
- `ip route 0.0.0.0 0.0.0.0 <IP> weight 1 enable` — wrong VOSS syntax (needs CIDR + two steps)

### How to discover IPE lan1 IP:
```
show ip arp vrf GlobalRouter    ← look for ARP entry on VLAN 4047 / Port 1/1
show ip interface               ← check if VIQ gave VLAN 4047 an SVI IP
```

### Correct transit block (once IPE lan1 IP known):
```
ip route 0.0.0.0/0 <IPE_lan1_IP> weight 1
ip route 0.0.0.0/0 <IPE_lan1_IP> enable
```
IP Shortcuts: `router isis` → `ip-shortcut enable` (makes SW2 a fabric internet exit)

## VOSS CLI Key Syntax (confirmed working)
```
ip route 0.0.0.0/0 192.168.0.1 weight 1    ← create
ip route 0.0.0.0/0 192.168.0.1 enable      ← activate (separate step)
ip name-server primary 8.8.8.8
vlan members add 1 1/1
sys name SW2-VOSS
save config
```

## Open Questions
| # | Question | Status |
|---|----------|--------|
| Q1 | What is the IPE? | ✅ Physical Extreme Networks appliance |
| Q2 | RDC tunnel protocol (IPsec/FE/SPB/GRE)? | ❓ OPEN — RDC pre-configured, need their params |
| Q3 | Service offerings at RDC Raleigh? | ❓ OPEN |
| Q4 | VOSS upgrade path? | ✅ XIQ push, 9.3.2.0.GA |
| Q5 | Fabric scope (full SPB vs simple)? | ❓ OPEN |
| Q6 | New IP addressing? | ✅ See topology above |
| Q7 | XIQ or XMC? | ✅ XIQ via ZTP+ |

## Phases
| Phase | Name | Status |
|-------|------|--------|
| 0 | Architecture Design | ✅ Q1/Q4/Q6/Q7 done; **Q2 resolved May 4: GRE tunnel**; Q3/Q5 open |
| 1 | SW2 VOSS Install | ✅ Done Apr 30 |
| 1b | SW2 XIQ Onboard (clean ZTP+) | ✅ Done — KhKLab-SW-01 live in VIQ |
| 2 | IPE Deployment + RDC Connectivity | ✅ **SW2↔IPE↔RDU DC live (confirmed May 7)** |
| 3 | VOSS Fabric / SVIs / DHCP Config | **IN PROGRESS — Today (May 7)** |
| 4 | AP2 + XIQ/XMC Integration | After Phase 3 |
| 5 | RDC Services Integration | Later |
| 6 | E2E Test + Documentation | Later |

## Port Plan (re-confirmed May 7)
- **Port 1/1** → IPE (uplink to RDU DC, operational)
- **Port 1/3** → AP2 (AP3000)
- **Port 1/5** → MacBook / corporate user
- VLANs to deploy today: **Corp_New (70/I-SID 100070)** + **Guest_New (80/I-SID 100080)**

## Tomorrow's Plan (parked from May 7 evening, reopen 2026-05-08)
**Two-part deliverable user requested:**
1. **Outline EXOS→VOSS 4-principle cheatsheet** — see `reference_exos_voss_4_principles.md` (Brain/Muscle/Service/Edge) + `reference_voss_standards_rfcs.md` (the underlying IEEE/RFC standards)
2. **Walk through each principle to configure SW2** with Corp + Guest VLANs via IPE → RDC Raleigh — Socratic mode (no pre-cached CLI)

**User-stated topology variant:** "SW2_VOSS is going through my IPFIRE to connect with the RDC in Raleigh."
- **Open question:** Is "IPFIRE" = the Extreme IPE-40AX (`KhKLab-IPE40AX`) or a SEPARATE IPFIRE-software-based firewall in the path? Earlier memory says path is SW2 1/1 → IPE LAN1 → IPE WAN2 → QuantumFiber → RDC. Need to confirm tomorrow whether IPFIRE is just a colloquial name for IPE or an additional appliance.

**Port plan ambiguity to confirm:** User wrote "port-2 to AP2" in May 7 evening message but earlier confirmed "AP2 on 1/3" same day. NLLM's recipe also assumed 1/2. **Use 1/3 unless user re-corrects on Monday.**

## NLLM "Memory Lock" snapshot (May 1 EOD, surfaced May 7)
Per NotebookLLM/Alex synthesis 2026-05-07 17:48, these facts were locked at last work session (May 1) and confirmed verified:
| Element | Value | Status |
|---------|-------|--------|
| **System-ID** | `0000.0000.0002` | ✅ Locked May 1 |
| **Manual area** | `49.b0b1` | ✅ Locked May 1 |
| **Nick-name (SPB Fabric Passport)** | `0.00.02` | ✅ Locked May 1 (verified via `show isis spbm` 19:42 EDT) |
| **B-VIDs (backbone VIDs)** | `4051` (primary) + `4052` | ✅ Verified active |
| **Service VLANs** | VLAN 70 Corp_New + VLAN 80 Guest_New | ✅ Created May 1 |
| **I-SID mapping** | 70→100070, 80→100080 | ✅ Mapped May 1 |
| **DHCP pool — Corp** | 10.70.0.100–200, GW 10.70.0.1 | ✅ Defined |
| **DHCP pool — Guest** | 10.80.0.100–200, GW 10.80.0.1 | ✅ Defined |
| **VLAN 4047 (SD-WAN/IPE transit)** | Identified as VIQ-assigned | ⏳ Verify Monday |
| **IPE LAN1 IP candidate** | `192.168.0.25` (NLLM hypothesis) | ⏳ Verify Monday — earlier memory had this UNKNOWN |

**NLLM's recipe also produced a verbatim CLI block for SW2 deployment.** Per Socratic agreement (logic before CLI), it is intentionally NOT cached in memory. User can refer to `/Users/khukhan/Downloads/NoteBookLLM_EXOS-VOSS_TidBits_May 7 2026.docx` if needed as a sanity-check baseline AFTER deriving the answer Socratically.

**User said they'll share current EXOS configs as input** — wait for those before writing the SW2 VOSS config. SW1 EXOS tech-support is already in repo at `docs/data/may7-dhcp/tech_support_SW1_EXOS_May7_2026.txt` (1.07 MB) — has full `show configuration` output. Can extract from there if user doesn't paste a new dump.

**NotebookLLM/Alex 4-principle cheatsheet (verbatim source):**

| Principle | EXOS Logic (VLAN-Centric) | VOSS Logic (Service-Centric) |
|-----------|---------------------------|------------------------------|
| 1. The Brain | STP / OSPF: manually define every path; block redundant links | IS-IS: silent control plane; builds loop-free map of every possible path |
| 2. The Muscle | 802.1Q Trunks: manually tag every VLAN on every uplink | SPB (802.1aq): MAC-in-MAC "suitcases" move across the core automatically |
| 3. The Service | VLAN ID: locally significant; align manually across switches | I-SID: 24-bit global service ID; stretches services across Fabric automatically |
| 4. The Edge | Static Config: hard-coded access/trunk with specific tags | Auto-Sense / FA: ports detect devices (e.g., AP2) and dynamically provision |

## Hardware
### KhKLab-SW-01 (SW2 VOSS)
- Model: 5320-16P-2MXT-2X-FabricEngine
- Serial: FJ012544G-00483, MAC: d8:e0:16:3b:54:00
- OS: VOSS FabricEngine 9.3.2.0
- CLIP mgmt IP: 10.158.4.1/32
- Console: /dev/cu.usbserial-A9VKJO11 115200, login: FB-SNMP (also rwa)

### KhKLab-IPE40AX (IPE — CONFIRMED)
- Model: ipe-40ax-V2 (Extreme Networks)
- Serial: A0925V0322A0
- Firmware: 24.5.4.1
- Mgmt IP: 172.16.0.57 (on 172.16.x.x subnet)
- Site: KhKLab Spoke
- Role: Spoke (Spoke-Hub topology — RDC Raleigh is Hub)
- Tenant/Network: RDU-FishBowl (ExtremeCloud SD-WAN header)
- State: Managed / Active
- Fabric switch peering: KhKLab-SW-01 — Status: Active, 1 peer up
- Managed by: XIQSE (on-prem) at 10.152.0.110
- Orchestrator: Connected, config in-sync as of 2026-05-01 10:09:11
- Physical: IPE LAN1 → SW2 Port 1/1, IPE WAN2 → QuantumFiber

### IPE Interface Throughput (ExtremeCloud SD-WAN Dashboard — May 1 2026)
| Interface | State | Throughput | Connected To |
|-----------|-------|------------|--------------|
| lan1 | Active (green) | 25.18 kbps | SW2 Port 1/1 (VLAN 4047 SD-WAN) |
| lan2 | Inactive | 0 bits | Not connected |
| wan1 | Inactive | 0 bits | Not connected |
| wan2 | Active (green) | 59.96 kbps | QuantumFiber modem (WAN path) |
| wan3 | Inactive | 0 bits | Not connected |
- wan2 is the active internet path → QuantumFiber → Internet → VIQ cloud
- lan1 confirms IPE↔SW2 fabric link is active and passing traffic

### Management Architecture (confirmed May 1)
- XIQSE (on-prem NMS): 10.152.0.110 — manages IPE + SW2
- VIQ (cloud): also connected
- DNS: 10.152.0.11, domain: RDUFishbowl.local
- All devices on 10.152.x.x / 10.158.x.x management networks

### AP2
- Model: AP3000, IP: 192.168.0.25
- Connected: SW2 Port 1/3, FA active (clientWapType1)
