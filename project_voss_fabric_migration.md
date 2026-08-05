---
name: VOSS/FabricEngine Migration Project — Full State
description: EXOS→VOSS migration for Horizon lab. voss_migration_horizon.html LIVE on GitHub Pages (Apr 28 2026). 35 sidebar sections: 14 original + 12 Masterclass slides (M1-M12) + 9 Heart of VOSS (D1-D9). Pending next revision: add RFC/IEEE hyperlinks to D1.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Project Location
GitHub: https://github.com/khursheedkhanaiforgood-ai/voss-fabric-migration
GitHub Pages: https://khursheedkhanaiforgood-ai.github.io/voss-fabric-migration/ (enable Pages in settings)
Local: /Users/khukhan/voss-fabric-migration

## Source Lab (EXOS baseline — already working)
- SW1: 5320-16P-2MXT-2X @ 192.168.0.28 — VLANs 20+30 (Corp/Guest) — AP3000 on port 3
- SW2: 5320-16P-2MXT-2X @ 192.168.0.11 — VLANs 50+60 (Corp2/Guest2) — AP3000 on port 3
- VLAN 10: SW_VLAN_10_Mgmt — SVIs 10.10.0.1/10.10.0.2
- Reference: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/

## Key Feasibility Facts (from Google AI dialogue)
- 5320 is "Universal Hardware" — dual-persona capability: can run EXOS or VOSS
- OS change via: Boot menu (spacebar → "Change OS to VOSS"), EXOS CLI wizard, or XIQ (Actions → Change OS)
- OS change is DESTRUCTIVE — wipes all config, logs, events
- 5320 VOSS LIMITATIONS: NO vIST support, NO stacking (stacking = EXOS only)
- Extended Temp models (5320-24T-4X-XT, 5320-24T-24S-4XE-XT): EXOS ONLY — cannot convert

## Chosen Architecture — Option A: Resilient Fabric Core
- NNI (Port 17) connects SW1↔SW2 — IS-IS/SPB fabric
- Both switches connect to Quantum Fiber modem (Port 1) — redundant internet exit
- AP3000 on Port 3 each switch — Fabric Attach (auto-provisioning)
- Anycast Gateways — same IP (10.0.X.1) on both switches simultaneously
- IP Shortcuts — SW2 learns SW1's default route via Fabric (no manual route on SW2)

## Physical Port Mapping (5320-16P-2MXT-2X)
- Port 1: Quantum Fiber modem (internet exit)
- Port 3: AP3000 (PoE+ powered)
- Port 17: NNI / Fabric backbone (Multi-Gig RJ45 — recommended over SFP+ ports)

## VLAN → I-SID Service Map (convention: VLAN ID + 100,000)
| Service | VLAN | I-SID  | IP Gateway (Anycast) | QoS Priority |
|---------|------|--------|----------------------|--------------|
| MGMT    | 10   | 100010 | 10.0.10.1            | 6            |
| Alpha   | 20   | 100020 | 10.0.20.1            | 6            |
| Bravo   | 30   | 100030 | 10.0.30.1            | 2            |
| Delta   | 50   | 100050 | 10.0.50.1            | 1            |
| Gamma   | 60   | 100060 | 10.0.60.1            | 1            |

## SPBM Control Plane Parameters
- Ethertype: 0x8100 (MUST match on both switches — most common adjacency failure cause)
- Manual Area: 00.0001 (MUST be identical on both)
- SW1: system-id 0000.0000.0001, nick-name 0.00.01
- SW2: system-id 0000.0000.0002, nick-name 0.00.02

## VOSS CLI — SW1 (Gateway Switch) Ready-to-Paste
```
enable
configure terminal
# --- Global Fabric Setup ---
router isis
 system-id 0000.0000.0001
 manual-area 00.0001
 exit
spbm 1
 spbm ethertype 0x8100
 spbm nick-name 0.00.01
 exit
router isis enable
# --- NNI Link (Port 17) ---
interface gigabitEthernet 1/17
 isis enable
 isis spbm 1
 no shutdown
exit
# --- Service Provisioning ---
vlan create 10,20,30,50,60 type port-mstprstp 0
vlan i-sid 10 100010
vlan i-sid 20 100020
vlan i-sid 30 100030
vlan i-sid 50 100050
vlan i-sid 60 100060
vlan name 10 "MGMT"
vlan name 20 "Alpha"
vlan name 30 "Bravo"
vlan name 50 "Delta"
vlan name 60 "Gamma"
# --- Anycast Gateways + DHCP ---
interface vlan 10
 ip address 10.0.10.1/24
exit
interface vlan 20
 ip address 10.0.20.1/24
 ip dhcp-server enable
exit
interface vlan 30
 ip address 10.0.30.1/24
 ip dhcp-server enable
exit
interface vlan 50
 ip address 10.0.50.1/24
 ip dhcp-server enable
exit
interface vlan 60
 ip address 10.0.60.1/24
 ip dhcp-server enable
exit
# --- DHCP Pools (SW1 = authoritative DHCP server) ---
ip dhcp-server enable
ip dhcp-server pool Alpha
 network-address 10.20.0.0 255.255.255.0
 range 10.20.0.100 10.20.0.200
 default-router 10.20.0.1
 dns-server 8.8.8.8 8.8.4.4
 exit
ip dhcp-server pool Bravo
 network-address 10.30.0.0 255.255.255.0
 range 10.30.0.100 10.30.0.200
 default-router 10.30.0.1
 dns-server 8.8.8.8 8.8.4.4
 exit
# (repeat for Delta/Gamma pools on 10.50/10.60)
# --- AP Port (Port 3 — Fabric Attach) ---
fa enable
interface gigabitEthernet 1/3
 auto-sense enable
 fa enable
 qos 802.1p-override disable
 no shutdown
exit
# --- Internet Exit (Port 1 → Modem) ---
vlan create 100 name "Internet_Exit" type port-mstprstp 0
vlan members add 100 1/1
interface vlan 100
 ip address 192.168.1.2 255.255.255.0
exit
ip route 0.0.0.0 0.0.0.0 192.168.1.1 weight 1 enable
router isis
 ip-shortcut
exit
# --- QoS for Voice ---
vlan config 20 default-control-priority 6
save config
```

## VOSS CLI — SW2 (Remote Switch) — differences only
- system-id 0000.0000.0002, nick-name 0.00.02
- interface vlan 100: ip address 192.168.1.3 (not .2)
- Same I-SID/VLAN block (must match SW1 exactly)
- No DHCP pools on SW2 — SW1 is authoritative; Fabric relays DHCP across NNI

## Key EXOS→VOSS CLI Differences
| Concept | EXOS | VOSS |
|---------|------|------|
| IP forwarding | `enable ipforwarding vlan X` | IMPLICIT — assigned IP = routing enabled |
| DHCP server | `configure vlan X dhcp-address-range` + `enable dhcp ports 3 vlan X` | `ip dhcp-server enable` + pool + `ip dhcp-server enable` on interface vlan |
| Default route | `configure iproute add default <IP>` | `ip route 0.0.0.0 0.0.0.0 <IP> weight 1 enable` |
| Save config | `save configuration` | `save config` |
| VLAN trunking | Manual tag every VLAN on every uplink | Zero config on NNI — I-SIDs flow automatically |
| AP port | `configure vlan X add ports 3 tagged` | `auto-sense enable` + `fa enable` |
| Inter-switch routes | Static routes on each switch | `ip-shortcut` under router isis |

## Verification Commands (VOSS)
```
show isis adj              # State must be UP (L1-2) — backbone check
show isis interface        # NNI port OperState must be Up
show spbm                  # Ethertype must be 0x8100 on both switches
show i-sid                 # 5 VLANs listed, Type=ELAN
show fa neighbor           # AP3000 visible on Port 1/3
show fa assignment         # VLANs dynamically mapped — State: ACTIVE, Type: DYNAMIC
show ip route              # 0.0.0.0/0 via 192.168.1.1 (SW1) or via SW1 nick-name (SW2)
show ip dhcp-server summary
show ip dhcp-server binding
show application iqagent status  # Connection: Connected, Server: hac.extremecloudiq.com
show isis spbm i-sid       # Confirms East-West service visibility
show isis spbm unicast-tree # Shows exact physical path traffic is taking
```

## Troubleshooting Decision Tree
**No ISIS adjacency:**
1. `show spbm` → ethertype 0x8100 on BOTH (mismatch = silent drop)
2. `show isis` → manual-area identical on both
3. `show isis` → system-id unique (same ID = adjacency flaps)
4. `show isis interface` → OperState Up on NNI port
5. `debug isis adjacency` → see if Fabric hellos are being sent
6. MTU: NNI port must be ≥1522 (MAC-in-MAC overhead)

**AP not getting VLANs:**
1. `show fa neighbor` → if empty, LLDP blocked
2. `show interface gig 1/3 verbose` → check auto-sense enabled
3. `show vlan i-sid` → I-SID must exist before switch grants FA request
4. Toggle: `no fa enable` then `fa enable` on interface to force re-sync

**No internet / 169.254.x.x (APIPA):**
1. `show ip dhcp-server summary` → server enabled?
2. `show fa assignment` → VLAN activated on port?
3. `show ip route` → default route present?
4. `ping 192.168.1.1` → switch-to-modem L2 working?

## EXOS Backup (Do BEFORE OS change)
```
# On each EXOS switch:
save configuration as-script switch1_backup.xsf
cp /usr/local/cfg/switch1_backup.xsf /usr/local/ext/
# Verify USB
ls /usr/local/ext
show file /usr/local/ext/switch1_backup.xsf
unmount /usr/local/ext   # safe eject — MUST do before removing USB
```
XIQ: Manage > Devices > Select switch > Backups > Create Backup
XIQ: Configure > Network Policies > Clone (name it EXOS_Pre_Migration_Backup)
AP config: Manage > Devices > Configuration Audit icon > Complete tab → copy to file

## Standards Governing VOSS/Fabric Connect
- **IEEE 802.1aq** — SPB (Shortest Path Bridging) — control plane architecture
- **RFC 6329** — IS-IS Extensions for SPB
- **IEEE 802.1ah** — Provider Backbone Bridging (MAC-in-MAC) — data plane
- **IEEE 802.1Qcj** — Fabric Attach standard
- **IEEE 802.1ag** — OAM (Connectivity Fault Management)

## XIQ for VOSS (Zero-CLI method)
1. Manage > Devices > Actions > **Change OS → VOSS** (triggers reboot + image)
2. Network Policy > Switching > Common Settings → Enable Fabric Connect (SPBM), set Manual Area
3. Device Template: Port 17=NNI, Port 3=Auto-Sense+FA, Port 1=Access (Internet VLAN 100)
4. Fabric Attach section: map VLAN ID → I-SID for each SSID
5. Perform **Complete Configuration Update** to push all settings

## Fabric Attach / AP3000 Integration
- AP is "thin" — pulls all intent from XIQ Network Policy
- AP uses LLDP-FA TLVs to request VLANs from switch
- Switch must have `fa enable` (global) + `auto-sense enable` + `fa enable` (port 3)
- Native/management VLAN: keep as VLAN 1 for initial discovery; FA handles tagged data VLANs
- XIQ: Configure > Network Policies > AP3000 Template > Wired Interface (Eth0) > Fabric Attach > map VLAN→I-SID

## Failover Behavior (Option A)
- NNI cable unplugged: 0-1 dropped pings (vs 3-10 with STP/EXOS)
- Traffic shifts to local modem link via Anycast Gateway
- `show isis adj` → adjacency DOWN; `show ip route` → 0.0.0.0 now via local modem
- Recovery: plug cable back → IS-IS auto-reconnects, traffic moves to optimal path

## Digital Twin System Definition (for Claude/NotebookLM)
Use the Master Lab Guide from the dialogue as context. Nodes:
- Node_01 (SW1): nick-name 0.00.01, system-id 0000.0000.0001
- Node_02 (SW2): nick-name 0.00.02, system-id 0000.0000.0002
- Node_03/04 (APs): FA clients on Port 1/3
- Node_05 (Gateway): Modem 192.168.1.1 on Port 1/1

## Phase Status
- Phase 1 (EXOS backup + OS conversion): PLANNED
- Phase 2 (SPB fabric + VLAN/I-SID): PLANNED
- Phase 3 (DHCP + routing + AP verify): PLANNED
- All 4 SSIDs must work on iPhone before migration is complete

## Why: Context
User completed EXOS lab Apr 7 2026. Wants to explore enterprise-grade Fabric Connect (SPB/VOSS) on same hardware to understand service-centric vs hop-by-hop networking model. VOSS is the more advanced persona — used in large enterprise and carrier deployments.

---

## Horizon Lab Variant — Port 10 NNI (Apr 22 2026)

Source: NoteBook_LLM_EXOS-VOSS_APR 22 2026.docx — Half 2 (paras 200+)
This variant applies VOSS to the **Horizon lab physical layout** where the inter-switch trunk is **Port 10** (not Port 17 from the April 8 architecture). Same switches, same I-SIDs, different NNI port.

### Port Translation — EXOS → VOSS (Horizon Layout)

| Port | EXOS | VOSS |
|------|------|------|
| Port 10 | Manual trunk — tag every VLAN | NNI: `isis enable` + `isis spbm 1` — I-SIDs flow automatically |
| Port 3 | Manual tagged trunk (VLAN 20/30) | Fabric Attach: `auto-sense enable` + `fa enable` — AP requests VLANs dynamically |
| Port 5 | Static access port | UNI: `vlan members add 10 1/5` |
| Port 1 | Default VLAN untagged to modem | Transit VLAN 100 (SW1 only) → default route to modem |

### CLI Scripts — Horizon Port 10 Variant

**SW1 (Gateway):**
```
enable
configure terminal
router isis
system-id 0000.0000.0001
manual-area 00.0001
exit
spbm 1
spbm ethertype 0x8100
spbm nick-name 0.00.01
exit
router isis enable
vlan create 10,20,30,50,60 type port-mstprstp 0
vlan i-sid 10 100010
vlan i-sid 20 100020
vlan i-sid 30 100030
vlan i-sid 50 100050
vlan i-sid 60 100060
# Port 10 — NNI
interface gigabitEthernet 1/10
isis enable
isis spbm 1
no shutdown
exit
# Port 3 — Fabric Attach
interface gigabitEthernet 1/3
auto-sense enable
fa enable
no shutdown
exit
# Port 5 — Wired client
vlan members add 10 1/5
# Port 1 — Internet exit (Transit VLAN 100)
vlan create 100 name "Internet_Exit" type port-mstprstp 0
vlan members add 100 1/1
interface vlan 100
ip address 192.168.1.2/24
exit
ip route 0.0.0.0 0.0.0.0 192.168.1.1 weight 1 enable
router isis
ip-shortcut
exit
save config
```

**SW2 (Edge — differences only):**
- `system-id 0000.0000.0002`, `nick-name 0.00.02`
- Same VLAN/I-SID block (must match SW1 exactly)
- Same Port 10 NNI + Port 3 FA + Port 5 UNI
- `ip-shortcut` enabled — learns default route from SW1 via Fabric
- No DHCP pools on SW2

### Anycast Gateway (both switches — same IP)
```
interface vlan 10
ip address 10.0.10.1/24
exit
interface vlan 20
ip address 10.0.20.1/24
exit
interface vlan 30
ip address 10.0.30.1/24
exit
interface vlan 50
ip address 10.0.50.1/24
exit
interface vlan 60
ip address 10.0.60.1/24
exit
```
Replaces VRRP. Both switches share identical IP+MAC per VLAN. Sub-second failover — no VRRP priority management needed.

### IP Shortcuts
```
router isis
ip-shortcut
exit
```
SW2 learns `0.0.0.0/0` from SW1 automatically via IS-IS. Verify on SW2: `show ip route` → next hop = SW1 nick-name (0.00.01), Protocol = SPB/ISIS.

### Failover Test Procedure
1. From MacBook on SW2: `ping 10.0.20.1 -t` (Anycast) + `ping 8.8.8.8 -t` (internet)
2. Physically unplug Port 10 NNI cable
3. **Expected:** 0–1 dropped pings (vs 3–10 with EXOS STP)
4. Verify isolated state: `show isis adj` (adjacency DOWN), `show ip route` (0.0.0.0 now local), `show isis spbm i-sid` (I-SIDs = "Local Only")
5. Plug cable back → IS-IS auto-heals, traffic returns to optimal path

### XIQ Monitoring Post-Migration
- **Network 360** (ML Insights → Network 360 Monitor): SPB/Fabric link shown between switches — green = IS-IS healthy, red/orange = break
- **Client 360** (Manage → Clients → MAC): Topology strip shows wireless hop → Port 3 FA → NNI MAC-in-MAC tunnel → SW1 Port 1 → modem
- **Floor Plan**: Upload floor plan image, scale to known distance, drag AP3000s to position, toggle RSSI heat map per SSID (VLAN 20 vs VLAN 30)
- **Failover verification**: IS-IS Adjacency Down event visible in Manage → Events (filter: System/Critical)

### Key Student Insight (Alex, Apr 22)
"Instead of managing a complicated matrix of 'which VLAN is tagged on which port,' we just define the NNI link on Port 10 and the AP port on Port 3. The Fabric handles the rest of the plumbing behind the scenes."
"In our old EXOS setup, Switch 2 was just a 'dumb' follower. With Anycast, Switch 2 becomes a fully independent gateway — both switches share identical GPS and the authority to drive the car."

---

## Apr 28 2026 — HTML Reference Page BUILT + LIVE

### voss_migration_horizon.html — COMPLETE
**URL:** https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/voss_migration_horizon.html
**Local:** /tmp/main-work/voss_migration_horizon.html
**Linked from:** index.html, index-nyt.html, index-harpers.html — Training Resources section, cyan card labelled "EXOS→VOSS Migration"

**Page structure — 35 sidebar sections total:**

| Group | Sections | Source |
|-------|----------|--------|
| Original 14 | Overview, P0–GPS, SW1/SW2 CLI, V1/V2, R1/R2/R3 | Apr 22 + Apr 28 DOCX files |
| Masterclass M1–M12 | 12 slides rendered as HTML | VOSS_Migration_Masterclass_Horizon.pptx (NotebookLLM → PPTX, Apr 28) |
| Heart of VOSS D1–D9 | 9 sections from full Socratic dialogue | The heart of EXOS-VOSS from NoteBookLLM_Apr 22 2026.docx (462 paras) |

**D1–D9 content map:**
- D1: The Four Laws (802.1aq / RFC 6329 / 802.1ah / 802.1Qcj) — standards with explanations
- D2: I-SID Service Model — VLAN vs I-SID scope, Rosetta Stone table, full CLI
- D3: Port Translation — EXOS→VOSS for all ports, full SW1+SW2 scripts
- D4: Anycast — 4 failure scenarios without it (ping drop, roaming gaps, tromboning, complexity)
- D5: IP Shortcuts GPS check — verify `0.0.0.0/0 → 0.00.01` nick-name (not IP) on SW2
- D6: Fabric Failover Test — two-terminal procedure, isolated state checks, auto-heal
- D7: XIQ Monitoring — Event Logs, Network 360, Client 360 packet journey, Wireshark
- D8: XIQ Alerts + Webhooks — Horizon_Fabric_Alerts policy, VOSS XML-Notification CLI, Slack/Discord
- D9: Alex's Milestone — full synthesis, 6-row mental model table, CoPilot next frontier

**Source documents integrated (all 5):**
1. NoteBook_LLM_EXOS-VOSS_APR 22 2026.docx — Half 2, Port 10 NNI
2. ClaudeAI_EXOS-VOSS_Horizonz_Apr 28 2026.docx — 7-step build sequence
3. NoteBook_LLM_EXOS-VOSS_Apr 28 2026.docx — 9-step deployment table, "90%+Glue+GPS" framework
4. VOSS_Migration_Masterclass_Horizon.pptx — 12-slide NotebookLLM-generated deck (text-based, no images)
5. The heart of EXOS-VOSS from NoteBookLLM_Apr 22 2026.docx — 462-paragraph full Socratic dialogue

**Commits (Apr 28 2026):**
- 5ed5ff6 — initial page + all 3 index cards
- 087dc5e — Masterclass deck M1–M12 (+558 lines)
- 2faefea — Heart of VOSS D1–D9 (+502 lines)

### PENDING — Next Revision
**Add RFC/IEEE hyperlinks to D1 (The Four Laws section)**
- IEEE 802.1aq — https://ieeexplore.ieee.org/document/6047717
- RFC 6329 — https://www.rfc-editor.org/rfc/rfc6329
- IEEE 802.1ah — https://ieeexplore.ieee.org/document/4775741
- IEEE 802.1Qcj (Fabric Attach) — LLDP extension
- Why: User explicitly requested for "next revision" at end of Apr 28 session

**Why:** User wants the reference page to link to actual standards so each "Law" in D1 can be explored in detail.
**How to apply:** In next session, open voss_migration_horizon.html, find the D1 section, wrap each standard badge in an `<a href="..." target="_blank">` tag using the URLs above.
