---
name: VOSS/FabricEngine Migration Project — Full State
description: EXOS→VOSS migration for Horizon lab. voss_migration_horizon.html LIVE on GitHub Pages (Apr 28 2026). 35 sidebar sections: 14 original + 12 Masterclass slides (M1-M12) + 9 Heart of VOSS (D1-D9). Aug 27: new thread — 5320 reverted to EXOS, building UZTNA (Extreme Platform ONE Security) demo lab.
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
- Phase 1 (EXOS backup + OS conversion): PLANNED — EXOS_SwitchtoVOSS only
- Phase 2 (SPB fabric + VLAN/I-SID): PLANNED
- Phase 3 (DHCP + routing + AP verify): PLANNED
- All 4 SSIDs must work on iPhone before migration is complete

## Why: Context
User completed EXOS lab Apr 7 2026. Wants to explore enterprise-grade Fabric Connect (SPB/VOSS) on same hardware to understand service-centric vs hop-by-hop networking model. VOSS is the more advanced persona — used in large enterprise and carrier deployments.

---

## Aug 5 2026 — Topology Update (2-site, 2-switch, 2-AP)

### Revised Physical Topology
| Device | Name | Location | OS | Uplink | Status |
|---|---|---|---|---|---|
| EN_5320 #1 | VOSS_Switch_RDU | RDU_Fishbowl | VOSS (already) | IPE→SD-WAN→home lab | Password locked — pending Karl |
| EN_5320 #2 | EXOS_SwitchtoVOSS | Home lab | EXOS → migrate | CenturyLink modem (already configured) | Ready to convert |
| AP3000 #1 | AP_NPS | RDU | — | Connected to VOSS_Switch_RDU | Active — 802.1X/NPS lab |
| AP3000 #2 | AP_VOSS | Home lab | — | Will connect to EXOS_SwitchtoVOSS post-migration | Available |

### Revised Migration Scope
- **VOSS_Switch_RDU** — already running VOSS. Use as live reference once password recovered (Karl). Do NOT re-migrate.
- **EXOS_SwitchtoVOSS** — local migration target. Phases 1-3 apply here only.
- **AP_VOSS** — will connect to EXOS_SwitchtoVOSS after VOSS migration. Fabric Attach provisioning via EP1.
- **Internet exit** — CenturyLink modem already configured on EXOS_SwitchtoVOSS from prior labs. Port 1 transit VLAN 100 config carries forward.

### VIQ Assignments
| VIQ | Owner | Used For |
|---|---|---|
| RDU_Fishbowl VIQ | Karl | VOSS_Switch_RDU + AP_NPS (802.1X/NPS lab) |
| khukhan+lab@extremenetworks.com | Khursheed (personal) | EXOS_SwitchtoVOSS + AP_VOSS migration work |

### Pending Before Phase 1
- Recover VOSS_Switch_RDU password (Karl) — use as VOSS reference during local migration
- D1-D9 Socratic revamp + script-bank build (in progress Aug 5 2026)
- Confirm EXOS_SwitchtoVOSS current config backed up before OS conversion

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

### Aug 20 2026 — D1–D9 Curriculum COMPLETE + RFC Links DONE
- D1 RFC/IEEE hyperlinks added to voss_migration_horizon.html (commit eacc7ea). All four standard badges now clickable.
- Full D1–D9 Heart of VOSS Socratic curriculum completed in one sitting. EOD: docs/session_summary_20260820.html (commit 97914ea).
- Key new concepts solidified: Two-Push Rule (switch policy FIRST, then AP Network Policy), Auto-Sense device differentiation (FA TLVs = AP; no FA = laptop), campus fabric (no dedicated core), mixed-mode VOSS+EXOS boundary.

### Aug 21 2026 — Home Lab VOSS Start (NEXT SESSION)
**Plan:** Start physical VOSS home lab with two 5320s.
- 5320_fabric_se — already running VOSS (claimed by SE)
- 5320_Exos_EP1 — running EXOS, claimed by EP1; convert to VOSS first
**First tasks in sequence:**
1. Back up EXOS config on 5320_Exos_EP1 (XIQ backup + `save configuration as-script`)
2. Convert 5320_Exos_EP1 to VOSS (XIQ Actions → Change OS, or boot menu)
3. Configure SE VOSS Domain for 5320_fabric_se (match SPBM params)
4. Configure EP1 Network Policy → Fabric Connect for 5320_Exos_EP1
5. Cable NNI → verify IS-IS adjacency
6. Push AP Network Policy (SSIDs + I-SID mappings) to both AP3000s
**Key reference:** CLI scripts in this file above (Horizon Port 10 variant — matches actual lab topology: Port 10 = NNI, Port 3 = AP FA port)
**Constraint:** Two orchestrators (SE + EP1) — SPBM parameters must be manually aligned. Do NOT cable NNI until both switches are independently verified.

---

## Aug 22 2026 — VOSS_MyLab Troubleshooting — BLOCKED (user self-solving)

**Switch:** 5320-16P-2MXT-2X-FabricEngine, VOSS 9.4.0.0, IP 192.168.0.29, in-band management ONLY (no OOB MGT port).
**Session goal:** Restore IQAgent + AP3000 (AH-556680) on port 1/3, console-only.

### What Was Solved
- IS-IS/SPBM fabric: UP. I-SIDs 10010 (VLAN 10), 10020 (VLAN 20) confirmed.
- FA Authentication Succeeded (first time ever) — auto-sense MSG-AUTH key matched after AP reboot.
- `show i-sid` → `u:1/3` for I-SID 10010: VLAN 10 correctly set as egress-untagged on port 1/3.
- `show vlan members 10` → port 1/3 ACTIVE (FA dynamic — not STATIC).
- `show vlan members 4048` → EMPTY — auto-sense onboarding VLAN NOT catching port 1/3 traffic.

### What's Still Broken

**AP3000 (AH-556680) — orange LED, no DHCP:**
- AP factory-reset during session (EP1 server names now empty, using redirector.aerohive.com).
- AP `show interface mgt0`: IP=192.168.1.1 (factory fallback), Rx=0, Gateway=0.0.0.0.
- Switch DHCP leases=0 — AP DISCOVERs NOT reaching VLAN 10 DHCP server.
- Suspected root cause: **PVID mismatch** — FA sets VLAN 10 as egress-untagged (`u:1/3`) but PVID (ingress classification for untagged frames) may still be VLAN 1 (default). If PVID=1, AP's untagged DISCOVERs → VLAN 1 → no DHCP server → dropped silently.
- `show vlan members 1` (Aug 22 07:07): port 1/3 NOT in VLAN 1. Only port 1/1 is ACTIVE in VLAN 1.
- `show interface GigabitEthernet 1/3` (Aug 22 07:07): VLAN column = **"Tagged"** — port is in trunk/tagged mode. FA assigned VLAN 10 egress-untagged (`u:1/3`) but did NOT convert port to access mode. Untagged ingress frames from AP are classified by PVID — PVID value unknown, not shown in this output.
- **PVID is the suspected root cause.** If PVID ≠ 10, AP's untagged DHCP DISCOVERs land on wrong VLAN → no DHCP server → dropped. PVID command syntax in VOSS 9.4 not yet confirmed (tried `vlan ports 1/3 pvid 10` = invalid).
- AP CLI: `no interface mgt0 dhcp` = "Incomplete command" — correct IQ Engine DHCP renew syntax unknown.
- `show ip dhcp-server statistics` = invalid command in VOSS 9.4.

**IQAgent — offline (accepted limitation):**
- `ip name-server` does NOT feed IQAgent `/etc/resolv.conf` in VOSS (confirmed).
- No Linux shell access, no OOB MGT port — cannot manually fix resolv.conf.
- IQAgent will only reconnect if switch gets DHCP at boot (ZTP+) — not possible with static IP.
- Status: accepted as blocked until NNI reconnected and Karl assists, or hardware workaround found.

**NNI port 1/10:** Still unplugged — switch isolated from Fishbowl fabric.

### Final State (end of Aug 22 session)
- Port 1/3: Access mode confirmed (PVID=10) ✓ — was Tagged before fix
- AP still Rx=0, no DHCP lease after reboot
- `show ip dhcp-server` revealed two new suspects:
  1. **HA Status: Initialising** — DHCP server stuck, not serving. Mgmt-clip set to 10.0.0.2 but only Clip1 (10.0.0.1) exists → address mismatch may be keeping HA in init state.
  2. **Authoritative: disabled**
- `show ip interface`: Vlan10 10.0.10.1 UP ✓, routing fine
- `show ip dhcp-server subnet` NOT yet run — do this first next session to verify pool exists
- Fix to try: `no ip dhcp-server enable` → `ip dhcp-server enable` → check if HA goes Active
- Also: create Clip2 at 10.0.0.2 to resolve mgmt-clip mismatch, OR change mgmt-clip to Clip1 (10.0.0.1)
- NNI port 1/10 still unplugged. IQAgent still offline.

### Resume Next Session
1. `show ip dhcp-server subnet` — verify 10.0.10.0/24 pool exists
2. Restart DHCP server — `no ip dhcp-server enable` / `ip dhcp-server enable`
3. Fix mgmt-clip mismatch (10.0.0.2 vs Clip1=10.0.0.1)
4. Watch for DHCP lease on AP

---

## Aug 24 2026 — Decision: Clean Rebuild (EXOS → EP1 → VOSS) Instead of Live DHCP Fix

**Decision:** User opted to wipe and redo cleanly rather than chase the mgmt-clip/HA DHCP fix live. Rationale: wants lessons learned baked into initial config rather than patched live.

**New facts confirmed this session:**
- VOSS_MyLab and AP_3000_MyLab (AH-556680) have BOTH dropped off EP1 entirely — not cloud-visible. Consistent with IQAgent-offline finding from Aug 22 (static IP never populated resolv.conf). Means OS conversion back to EXOS MUST be done via console/boot-menu, not EP1 Actions → Change OS.
- **NNI port CONFIRMED 1/10 ↔ 1/10 (VOSS_SE side and VOSS_MyLab side) — verified via LSDB check by user.** Previously unconfirmed on the VOSS_SE side; now confirmed truth, not assumption.
- **KB gap found (Workato KB Search, Aug 24):** Extreme KB has a clean, confirmed console boot-menu procedure for EXOS→VOSS (console @ 115200 baud, hold spacebar during boot, select "Change the switch OS to VOSS"). It does **NOT** have a confirmed boot-menu procedure for the reverse (VOSS→EXOS) — only a vague reference in an internal presentation to a `.xos` image download + activation + reset from VOSS CLI, no exact syntax. Do not guess at VOSS-side CLI install commands — verify further before running on live hardware.

### Clean Rebuild Plan (current plan of record)
0. Capture `show running-config` on VOSS_MyLab before wipe (reference only — Aug 22 confirmed-working ISIS/SPBM/I-SID block).
1. Revert VOSS_MyLab to EXOS via console boot-menu (spacebar at boot) — try same menu first, may symmetrically offer "Change switch OS to EXOS" even though currently on VOSS. If not present, need to find exact VOSS CLI `.xos` install/activate command before proceeding (KB gap above).
2. Let switch ZTP+ discover EP1 fresh from clean EXOS boot (fresh DHCP should also fix the resolv.conf/IQAgent issue that blocked cloud visibility).
3. Convert back to VOSS via EP1 (Actions → Change OS → VOSS) — should work now that it's cloud-visible again.
4. Rebuild VOSS config: same SPBM/I-SID/FA as before, PLUS explicit DHCP server HA/mgmt-clip check baked in this time (see mgmt-clip alignment fix under Aug 22 section — this was the real root cause, not tagging).
5. Cable + configure NNI 1/10 ↔ 1/10 to VOSS_SE this time (previously left unplugged/standalone).
6. Before trusting NNI adjacency, confirm with Karl whether the EVE-NG disk failure / IS-IS adjacency loss on Fishbowl side (vBOB02-Headend/Kit, Aug 19-20, tracked in `project_voss_siteengine_lab.md`) is resolved — separate blocker, unrelated to local rebuild.
7. Verify end to end: `show isis adj`, `show fa neighbor`, AP3000 DHCP lease, IQAgent reconnect.

**Status: PAUSED — user stepped away before console reboot. Resume at Step 1 (console boot-menu check) when user returns.**

---

## Aug 24 2026 (later same session) — Step 1 BLOCKED: EXOS revert path dead-ends

**Pre-wipe backup completed successfully** — full `show running-config` + `show i-sid` + `show vlan members` + `show spbm` + `show isis` + `show isis lsdb` + `show ip route` captured via `screen -L` to `~/voss_mylab_running_config_20260824.txt`.

**Live config revealed 4 discrepancies vs. the documented plan (all still true, useful for whichever path is chosen next):**
1. **manual-area is `49.b0b1`, NOT `00.0001`** as written throughout this doc's CLI reference. If VOSS_SE's manual-area doesn't match `49.b0b1`, that alone blocks ISIS adjacency — check before any NNI rebuild.
2. **`show isis` → `Num of Interfaces: 0`** — no port has `isis enable`/`isis spbm 1` configured, not even 1/10. NNI was never configured at the interface level, not just unplugged.
3. **VLAN 20 ("Guest", i-sid 10020) has zero port members** — only VLAN 10/Staff (port 1/3) was ever active. Second SSID's VLAN was never live on this switch.
4. **`show isis lsdb` shows only this switch itself** (`khKlab-SW-02`, 3 self-originated LSP fragments, area "HOME") — no VOSS_SE entry. No live adjacency currently exists, consistent with #2.
5. **Mgmt-clip mismatch theory from Aug 22 REVISED**: both `loopback 1` (Clip1 = 10.0.0.1, ISIS source) and `mgmt clip` (10.0.0.2, in-band mgmt) exist and are both configured — not a missing/mismatched clip as originally suspected. `show isis` confirms `Inband Mgmt Clip Ip: 10.0.0.2` active. The real cause of the DHCP HA "Initialising" state from Aug 22 is still unresolved.

**Boot-menu check result:** Rebooted with `reset -y`, held spacebar, got the boot menu (`5320-16P-2MXT-2X Boot Menu 3.6.0.3`). Options were **only**: `VOSS: Default`, `VOSS: Rescue`, `Run Manufacturing Diagnostics`, `Reboot system`. **No "Change switch OS to EXOS" option exists on this hardware/bootloader.** Backed out via ESC without selecting anything (booted back into VOSS normally).

**KB search (Workato KB, 2nd query) confirms:** no verified CLI command exists either for VOSS→Switch Engine/EXOS image install/activate. KB only has the forward direction (EXOS→VOSS, both boot-menu and `download image` CLI methods) fully documented.

**User then confirmed: no EXOS (`.xos`) image file available locally either.**

### Conclusion: EXOS-revert path is DEAD-ENDED for now
Blocked on all three fronts — no boot-menu option, no documented CLI procedure, no image file. Not pursuable until an EXOS image is obtained (Extreme support portal, requires valid support contract/serial) AND a documented/verified reverse-conversion procedure is found.

### Next Decision Point (pending user choice)
Two live options as of end of this session:
- **(A) Get EXOS image + verified procedure first**, then resume the original clean-rebuild plan once both blockers are cleared.
- **(B) Abandon the OS round-trip entirely** and instead fix VOSS_MyLab in place on its current VOSS install: add the missing ISIS interface config on port 1/10 (was never configured, per finding #2), align manual-area with VOSS_SE (finding #1), and resolve the DHCP HA "Initialising" issue directly (Aug 22 troubleshooting, cause still open per finding #5) — no wipe needed at all.

**User rejected further Claude-driven diagnosis on this topic mid-session (see `feedback_voss_mylab_no_unsolicited_diagnosis.md`).**

---

## Aug 24 2026 (later same session, cont'd) — NNI reconnected, IQAgent DNS fix attempt failed

**User re-inserted port 1/10 on both VOSS_SE and VOSS_MyLab.** Fabric adjacency confirmed UP: `show isis lsdb` now shows 6 systems in area HOME — khKlab-SW-02, KhKLab-SW-01, TSLab-SW-01, FishBowl-Fabric_Core-5720-01, Fishbowl-SW-01 (plus VOSS_MyLab itself). This resolves finding #2/#4 above — NNI is live now.

**IQAgent reinstall attempted:** `no iqagent enable` → `software iqagent reinstall` (from Privileged EXEC, correct mode) → `iqagent enable`. Agent restarted but still failed to connect:
- `Unable to resolve [hac.extremecloudiq.com] hostname` (DNS)
- Unable to get XIQ IP via DHCP, DNS, or Redirector methods
- `show application iqagent status` → Connection Status: Disconnected
- Confirmed via direct test: `ping 8.8.8.8` succeeds (internet path fine), `ping hac.extremecloudiq.com` → "Unknown Destination" (DNS resolution specifically broken)

**Conclusion: reinstalling IQAgent did not fix DNS resolution.** Same root cause as Aug 22 (`ip name-server` doesn't feed IQAgent's resolver in VOSS; needs DHCP-at-boot/ZTP+, not possible with static IP) — confirmed still true after reinstall.

**Useful corrected CLI syntax found this session (Workato KB):**
- `show fabric attach elements` — NOT `show fa neighbor` (invalid on this build)
- `show isis lsdb` — NOT `show lsdb` (lsdb is nested under isis)
- `reset -y` — reboot/reset command
- `software iqagent reinstall` — must run from Privileged EXEC, not from inside `configure terminal > application` submode

**Status: PAUSED — user is looping in Karl. Do not proceed with further diagnosis on this topic unless explicitly asked (see feedback file). Resume when user says "resume VOSS_MyLab" or similar, or reports what Karl said.**

**Awaiting user decision on A vs B.**

---

## Aug 27 2026 — New Thread: 5320_MyLab reverted to EXOS, UZTNA build-out started

**Context shift:** Separate from the VOSS_MyLab/VOSS_SE fabric thread above (still paused, awaiting Karl). This is the same physical 5320 + AP3000, now back on **EXOS** (`5320_Exos_MyLab`), starting a fresh lab goal: demonstrate Extreme's Universal ZTNA — now branded **"Extreme Platform ONE Security"** — end to end.

**Sequence so far:**
1. Port 1 (uplink) + Port 3 (AP3000) both verified untagged/Default VLAN only — no tagging leakage (`show vlan Default` + `show ports 1,3 information`).
2. Basic SSID pushed and confirmed working — client got DHCP from home router (QF-Modem) via Default VLAN bridging, no switch-side DHCP needed yet.
3. CAPWAP "policy reverts to default after push" issue from a prior VOSS session investigated: rollback-timer theory ruled out (CAPWAP never actually dropped), device-level override ruled out (user confirmed none present). Leading theory for next session: resave/redeploy ("Complete Configuration Push") — untested.
4. UZTNA design finalized: **EAP-TTLS** (not EAP-TLS) chosen — no client certs to distribute. Open-SSID + CWP ruled out per KB: CWP requires the SSID to be Enterprise 802.1X. RADIUS **Filter-Id** (single attribute) chosen over the 3-tuple Tunnel-Type method to map policy group → VLAN.
5. Policy `AP_EXOS_Aug27_UZTNA` built in EP1: Enterprise 802.1X + EAP-TTLS + CWP "UZTNA" layered on top, RADIUS Server Group configured, Filter-Id set. Password lives on the **individual local user record**, not the group — group membership only drives Filter-Id/VLAN assignment. SSID Name field still blank; 8 legacy/template profile rows (`UZTNA_Policy_TeamUSA→UZTNA_10`, `TeamAUS→UZTNA_20`, `VIP→UZTNA_40`, `Guest→UZTNA_30`, `POS→UZTNA_50`, plus `Test_UZTNA` entries) still present and unresolved.
6. **Explicit scope decision:** VLAN10/UZTNA_10 client rollout (port tagging + DHCP Supplemental CLI + home router return route) intentionally deferred — user wants DHCP working before authenticating any real client into that VLAN. Tonight's push stayed Wireless-only, no Switching/Routing changes.

**Drafted but NOT YET PUSHED CLI (for when DHCP rollout resumes):**
```
configure vlan UZTNA_10 dhcp-address-range 10.10.10.10 - 10.10.10.254
configure vlan UZTNA_10 dhcp-options default-gateway 10.10.10.1
configure vlan UZTNA_10 dhcp-options dns-server primary 8.8.8.8
configure vlan UZTNA_10 dhcp-options dns-server secondary 1.1.1.1
configure vlan UZTNA_10 dhcp-lease-timer 86400
enable dhcp ports 3 vlan UZTNA_10   # NOT idempotent
configure vlan UZTNA_10 add ports 3 tagged
```

**EOD published:** `docs/session_summary_20260827.html`, commit `3aac97a`, pushed to `voss-fabric-migration` main.

### Resume Next Session (tomorrow morning, per user)
1. Fill in SSID Name/Broadcast Name (still blank).
2. Check AP3000 IQ Engine firmware version.
3. Deploy Wireless-only policy push, watch CAPWAP for 15 min post-push (test the resave/redeploy theory from #3 above).
4. Decide what to do with the 8 legacy/template profile rows.
5. When ready to go live on VLAN10: tag port 3 + push the DHCP Supplemental CLI above + add home router return route.
6. Confirm Platform ONE Security/UZTNA subscription entitlement (KB flagged as "available with an additional subscription" for native cloud PKI — verify this lab's tenant has it).

**Status: PAUSED — user ending session for the night. Resume when user references this thread or says "continue UZTNA build."**

---

## Sept 1 2026 — New Thread: Fabric Extend (FE) Concept Study — PAUSED, user wants to study first

**Context:** Separate from both the VOSS_MyLab/VOSS_SE fabric thread and the EXOS UZTNA thread above. Same physical two 5320s (referred to here as SW1/home_lab and SW2/Karl's-managed-Voss_2), but this thread is pure architecture/concept discussion — no CLI run, no hardware touched.

**Also this session:** confirmed AP1's ETH0 uplink was set to Trunk in device-level EP1 setting (not a switch issue) — see `feedback_eth0_uplink_ep1_trunk.md`. EOD published: `docs/session_summary_20260831.html`, commit `1059406`.

**Sequence of concepts covered (KB-verified, not guessed):**
1. Fabric Attach (FA) vs Fabric Extend (FE) distinction — FA = LLDP-based dynamic per-port VLAN grant to an AP/host on an existing I-SID; FE = tunneling the SPB fabric itself over a routed IP WAN as a "virtual NNI," hub-and-spoke topology.
2. **Platform limitation confirmed via KB:** the user's 5320 switches support Fabric Extend via plain VXLAN (Fabric Engine 8.6+, well within their 9.4.0.0) but do **NOT** support the IPsec-encrypted variant of FE. IPsec-based FE requires a separate Fabric IPsec Gateway appliance, only available on 5720-24MXW/48MXW, 7520, 7720, VSP4900, VSP7400. This directly answers "can I do FE with an IPsec tunnel on my 5320s" — no.
3. Confirmed FE over a **private/dedicated WAN** (not raw public internet) is the documented-supported design — VXLAN tunnel, no IPsec needed. Raw public-internet-with-NAT VXLAN FE is NOT documented/confirmed either way for the 5320 — open question, not resolved.
4. Topology mapping confirmed: user's home_lab switch = hub/core, far-end switch (other side of dedicated WAN) = spoke/branch — matches FE's standard hub-and-spoke architecture, demonstrates "branch plugs in, seamlessly on network" use case.
5. **Tunnel termination point clarified (KB-verified):** the FE tunnel source IP terminates ON the VOSS switch itself (a brouter interface or loopback address, configured under `router isis` with `ip-tunnel-source-address`), NOT on the home_router. The router stays a plain L3 gateway/NAT box, unaware of FE — SW1 stays plugged into port 1 exactly as today. No separate VPN appliance exists for the plain-VXLAN (non-IPsec) path the 5320 uses — the Fabric IPsec Gateway appliance only exists for the unsupported IPsec variant.

**User then paused:** "I am a little confused about fabric extend/deployment. I need to study." — explicitly stopping to self-study rather than continue building on this. No further FE work should be pushed unprompted.

**Key open items for whenever this resumes:**
- Exact plain-VXLAN (non-IPsec) `logical-intf-tunnel`/`ip-tunnel-source-address` full CLI walkthrough — not yet looked up in full (only fragments found so far: `router isis` / `ip-tunnel-source-address <IP> vrf <vrf>` / `logical-intf isis <1-255> dest-ip <IP> name <word>`).
- Whether plain VXLAN FE reliably traverses raw public internet/NAT without a private WAN underneath — unconfirmed in KB.
- SW2/AP2 "reclaim from Karl's SiteEngine/IQ.Controller into MyLabVIQ" — not investigated.
- Phase 1 SW1 EXOS→VOSS lab build (backup → convert → Test 1/2/3 DHCP/FA staged tests) — drafted CLI ready, not yet executed on hardware.

**Status: PAUSED — user wants to study Fabric Extend concepts independently. Do not proceed with further FE research or push toward Phase 1 execution unless user re-initiates.**

---

## Sept 2 2026 — EP1 vs Site Engine fabric-orchestration comparison (user re-engaged FE thread via management-plane angle)

**Context:** User asked Claude to KB-verify whether EP1 can handle all fabric orchestration (incl. BCB/BEB provisioning) vs. Site Engine, then connected it to their own topology: SD-WAN/IPE connects to a VOSS switch acting as **BEB inside Karl's network**, and that switch is managed by **Site Engine**. This is a resumption of the paused FE thread via a new angle (management plane, not tunnel CLI) — treated as concept study, not a request to execute/build.

**KB-confirmed findings:**
1. **Site Engine has documented native fabric provisioning depth that EP1 does not yet match**: auto persona-switch (Switch Engine OS → Fabric Engine OS), topology config, L2VSN/L3VSN + Service ID/Name/Type (I-SID) config, DVR + redundancy protocols (VRRP/RSMLT/DVR), port templates, and visualization of IS-IS areas/Fabric Connect links/primary-secondary paths/service location. EP1's fabric story today = visualization + AI-assisted workflows layered on Fabric Connect's built-in automation, but no KB source shows a dedicated EP1 BCB/BEB or I-SID/VSN provisioning workflow. Extreme's own positioning: Site Engine = today's fabric-management depth; EP1 = strategic cloud/AI destination.
2. **Client does NOT need both products deployed.** KB confirms 3 supported modes: Site Engine only (local/on-prem or fully air-gapped, license-proxy-only option), EP1 only (cloud/SaaS-centric), or both together in "connected mode" for phased migration/mixed environments (e.g., Site Engine for switches, cloud for APs). Switching between air-gapped ↔ connected mode is a few clicks, no hardware/firmware change, no new subscription.
3. **CLI provisioning remains valid regardless of which management tool (if any) is in use** — consistent with how this lab has operated so far.
4. **Site Engine + SD-WAN/FE integration is a documented, supported combination**: Site Engine can display tunnels extending fabrics through SD-WAN, report tunnel failures between SD-WAN devices, and let the operator jump from Site Engine into the SD-WAN appliance's 360 view to troubleshoot. New FE mechanism detail (from SD-WAN advanced config deck): **SD-WAN allocates a /29 subnet per site from the Fabric Extend IP network**, and IPEs use **LLDP** to hand the Fabric switches their IP/gateway info + the destination IPs needed to auto-build the VXLAN tunnels — i.e., the FE tunnel addressing is SD-WAN-automated, not manually configured on the VOSS switch in this model.

**Open item (unconfirmed in KB, flagged rather than guessed):** the exact IP-reachability path Site Engine itself needs to manage a remote BEB switch sitting behind SD-WAN/IPE at Karl's network — i.e., whether Site Engine reaches the switch's mgmt IP in-band over the same FE/SD-WAN tunnel, or needs a separate OOB path. Not stated explicitly in any retrieved source. Would need direct testing or an Extreme docs/GTAC follow-up to confirm.

**Status: user is studying/validating the mental model, not requesting a hands-on build.** Do not push toward Phase 1 hardware execution or further unprompted FE research — continue only when user asks a further question or explicitly says to resume building.

### Same session, extended to a hypothetical client scenario: EP1-only (no Site Engine) Cisco→Fabric hub-and-spoke hospital migration

User proposed a professional/client-facing narrative: HQ + many remote sites (hospital), migrating from Cisco, using **EP1 only** (no Site Engine) — story = CLI builds core+BEB, then EP1 takes over ongoing management.

**KB-confirmed, refines but does not overturn the user's narrative:**
- Extreme's own documented **hub-and-spoke Fabric Extend reference architecture** (User Guide + Advanced Presentation deck) is built around **Site Engine / "Fabric Extend Manager"**, not EP1 — Main/Hub site (e.g., 2 IPEs) + Branch spokes (e.g., 1 IPE), using VSP7400/VSP4900/5720/Fabric IPsec Gateway/XA1400-series hardware. **This means "EP1 takes over" is accurate for ongoing visualization/AI-ops once the fabric exists, but the FE tunnel provisioning/management layer itself is documented as a Site Engine capability, not an EP1 one** — going EP1-only likely means CLI (not a GUI tool) also carries the FE tunnel build, not just BCB/BEB/VSN config.
- **IPsec reminder still applies**: if hospital remote sites need FE over public/uncontrolled WAN, IPsec-based FE requires the separate Fabric IPsec Gateway appliance (5720-24MXW/48MXW, 7520, 7720, VSP4900, VSP7400) — cheaper/smaller edge switches at spoke sites may not qualify; VXLAN-only FE needs a private/dedicated WAN underneath.
- **Cisco migration can be phased** — KB confirms "migrate at your own pace" messaging, and EP1 + Third-Party Management Engine (TPME) can manage existing Cisco gear centrally alongside new Extreme fabric during transition (useful for a "HQ core swapped first, spokes migrated over time, old Cisco visible in EP1 meanwhile" narrative).
- **Healthcare proof points found**: Concord Hospital (bridged legacy network with new Extreme fabric, zero downtime cutover), CHU Besançon (security + operational continuity), ZOL (schedule confidence, isolated/segmented environments) — usable as case-study backup for a hospital pitch.
- SPB fabric failover **sub-200ms** — relevant resiliency stat for a multi-site hospital story (life-safety systems tolerance for network reconvergence).

**Not a real build — scoped as a hypothetical/consulting narrative, not this lab's hardware.** No CLI executed, no memory of a new hardware project created.

**Follow-up: user confirmed hospital scenario has dedicated circuits (MPLS/leased line) from every remote site to HQ — resolves the IPsec question for this narrative.** KB confirms: on a private/dedicated WAN (MPLS, Ethernet, leased line), VXLAN-only Fabric Extend is fully sufficient — IPsec is not required because the SP WAN is already private with built-in security. Practical effect: the pricier **Fabric IPsec Gateway appliance requirement drops out of this design entirely** — any Fabric Connect-capable switch (5320/5420/5720/7520/7720 family) can serve as the branch-side tunnel endpoint directly, widening the switch choice at spoke sites. Separately noted: KB references an **ExtremeAccess Platform 1400 (XA1400)** appliance in SP-WAN branch datasheets — this appears tied to the SD-WAN/IPE overlay use case specifically, not a hard requirement for a straight dedicated-circuit VXLAN FE design; not yet confirmed whether it's optional or exclusive to the SD-WAN path for this scenario.

### Fundamentals clarified (same session): CLI-only single-remote-site migration, and terminology fix on IPE

**User confirmed the fork:** with a dedicated WAN, for a first "migrate one remote site" interop test, **neither Fabric Extend nor an SD-WAN/IPE appliance is required** — just deploy Fabric Engine on the clean VOSS switch, build SPB/IS-IS/I-SID via CLI, and connect its uplink to the existing dedicated-WAN circuit as a plain legacy VLAN edge handoff (per Extreme's own Best Practices deck: "start by replacing one existing switch with 'legacy' up-links to the rest").

**Terminology correction (user-initiated):** "IPE" is not a separate technology paired with SD-WAN — **IPE is the ExtremeCloud SD-WAN appliance hardware family itself** (KB-confirmed models: `ipe-30so` 250Mbps, `ipe-30ax` 500Mbps, `ipe-40ax-v2` 750Mbps, `ipe-420ax` 1Gbps — Branch-XS/M/DC-S tiers). SD-WAN is the service; IPE is the box that runs it. Corrected in `reference_ep1_extreme_platform_one.md`.

**Production open-internet path — two distinct architectures with different hardware implications (KB-confirmed):**
1. **Native FE-IPsec (no SD-WAN)** — the fabric switch itself terminates IPsec. Requires **Fabric IPsec Gateway** capability — only on 5720-24MXW/48MXW, 7520, 7720, VSP4900, VSP7400. The 5320 is excluded from this path (reconfirms Sept 1 finding).
2. **Fabric-over-SD-WAN (via IPE)** — KB explicit: *"Fabric Extend VXLAN tunnel forms over SD-WAN IPsec tunnels"* / "A single VXLAN tunnel is passing MAC-in-MAC based encapsulation for SPBM; Multiple IPsec tunnels provides resilience." Here the **IPE appliance itself supplies the IPsec overlay**, and Fabric Extend's VXLAN tunnel rides inside it — so the switch only needs VXLAN/FE capability (which the 5320 already has), not IPsec-Gateway-class hardware. **This means an IPE pair at each end is how a 5320 can participate in an open-internet FE deployment, sidestepping the Fabric-IPsec-Gateway hardware tier.**

**Caveat flagged, not asserted as fact:** KB confirms the stacking architecture (FE-VXLAN over SD-WAN-IPsec) but does not explicitly state this removes the Fabric-IPsec-Gateway restriction for the 5320 by name — this is a reasonable architectural inference, not a direct product statement. Recommend a docs/GTAC check before treating this as a production design decision.

**Correction (sharper primary-source check, same session): "no appliance in the path" is wrong — Fabric IPsec Gateway is always required for FE-over-IPsec, on every supported platform.** Fabric Engine 9.3 User Guide Table 120 (Fabric Extend product support):
- **5320: Not Supported** for FE-over-IPsec — full stop, not just "needs a gateway."
- 5720-24MXW/48MXW (FE 8.7+), 7520 (FE 8.10+), 7720 (FE 8.10+), VSP4900-12MXU-12XE/24XE (VOSS 8.3+), VSP7400 (VOSS 8.2+): all documented as **"Supported using Fabric IPsec Gateway"** — no platform shows native/appliance-free IPsec termination. Solution-brief marketing language ("the switch can use IPsec tunnels...") glosses over this; the User Guide table is authoritative.
- **New detail:** Fabric IPsec Gateway is a **VM** (2 CPU cores, 4GB RAM, 1 vport, min. 10GB SSD) providing FE tunnel aggregation + fragmentation/reassembly + IPsec encryption — not necessarily separate hardware. 5720 models need a physical SSD module installed to host it via Extreme Integrated Application Hosting.
- This sharpens (does not overturn) the earlier caveat: 5320's "Not Supported" is specifically for the native *Fabric-Extend-over-IPsec* feature (switch + Fabric IPsec Gateway VM pairing). Whether that also blocks the separate FE-VXLAN-over-independently-SD-WAN-encrypted-WAN architecture remains unconfirmed either way — still flagged as open, still recommend GTAC/SE confirmation before a production decision.

### RS1 cutover mechanics + migration sequencing (same session)

**Scenario:** Cut over one BEB at RemoteSite1 (RS1) + new EN APs, no NNI at this site (no other Fabric Engine neighbor yet), uplink runs to dedicated WAN or internet back to Cisco Core at HQ.

**Cisco-side config — confirmed required, but ordinary, not Fabric-aware:** KB explicit: *"the third-party core does not participate in Fabric or Fabric Attach signaling... requires conventional legacy configuration on the uplink, specifically matching VLANs at both ends of the link."* Cisco just needs a standard 802.1Q trunk (or routed peer) matching VLAN IDs — same as bringing up any new vendor switch. No Fabric-specific Cisco config exists or is needed.

**Two things on the RS1 box are separate:** (1) SPB/IS-IS/BVLAN + I-SID-to-port bindings = internal segmentation, valid to configure even with zero NNI neighbors (dormant until a peer exists). (2) The WAN-facing uplink to Cisco HQ = a plain UNI/legacy-edge VLAN-tagged port, NOT an NNI — unrelated to the BVLAN/IS-IS config.

**Migration sequencing — KB confirms no universal rule, validates a parallel-track approach:**
- Pattern 1 (edge-first, what RS1 already is): *"Start by replacing one existing switch with legacy uplinks to the rest. Then replace switches one at a time, connected to the first switch and so on."* Needs zero fabric core anywhere yet.
- Pattern 2 (parallel core): *"build the new fabric core in parallel to the existing core... trunk VLANs, keep routing on existing core initially, then move access and flip IP gateways gradually"* — explicitly called the gentle/gradual/reversible path.
- KB explicit: *"No universal rule that core must always be replaced first."*
- **Recommendation given (not yet acted on):** treat RS1 legacy-edge cutover and EN_Fabric_Core lab POC as parallel, non-blocking tracks — RS1 ships now as a standalone win; the core POC de-risks the later "join RS1 into a real fabric" step before touching live HQ Cisco core.

### Lab POC design: two-switch MyLab/RemoteLab, ZTF-vs-FE-both-ends-required correction (same session — still concept/planning discussion, no hardware touched)

**Scenario:** two VOSS switches at separate physical locations for a lab POC — BEB_VOSS_MyLab and BEB_VOSS_RemoteLab (RemoteLab has full SPBM/IS-IS/I-SID setup). User asked whether MyLab can stay "vanilla," inheriting ZTP/ZTF from RemoteLab, needing only new I-SIDs if any, with no SD-WAN/IPE between them.

**Correction confirmed by KB (2 targeted queries, cross-checked against a pasted "buddy" AI response making the same claim):** MyLab is **not** vanilla. Zero Touch Fabric/Auto-sense's zero-config inheritance is LLDP-driven, and LLDP is a local-link protocol — KB explicit: *"Auto-sense is a port-based functionality... based on LLDP events"* and separately, for the SD-WAN case, *"the Fabric switch uses LLDP subtype TLV 7 to tell the ipe about which logical IS-IS interfaces have formed adjacency"* — i.e. LLDP operates between a switch and its **directly attached** neighbor (physical port or local SD-WAN/IPE appliance), never end-to-end across the Fabric Extend IP tunnel itself. KB does not show LLDP/ZTF adjacency traversing the WAN tunnel.

**Both ends need explicit config — confirmed exact CLI sequence (Fabric Engine 9.3 User Guide):**
1. `ip-tunnel-source-address <local-IP> vrf fe`
2. `logical-intf isis <1-255> dest-ip <peer-IP> mtu <value> [name WORD]` — KB-sourced worked example uses `mtu 1500` on both sides (not the ≥1594 floor cited earlier this session as a minimum — 1500 appears in Extreme's own reference config, so treat 1594 as a safety margin claim needing reconciliation, not a hard KB-confirmed number)
3. `isis`, `isis spbm <1-100>`, `isis enable` on the logical interface
4. Same 4 steps mirrored on the peer switch, with source/dest IPs swapped — KB explicit: *"logical ISIS interface adjacency requires configuration on both sides."*

So both MyLab and RemoteLab need identical SPBM/IS-IS/BVLAN + FE-tunnel scaffolding — no shortcut where one side stays blank.

**Where the user's instinct WAS correct — I-SID provisioning specifically:** once the FE logical IS-IS adjacency is up, ordinary IS-IS LSDB flooding propagates existing I-SIDs automatically. MyLab does not need to redeclare RemoteLab's existing I-SIDs. MyLab only needs a **new** I-SID definition if it's originating a new service; it still needs a local `vlan i-sid` binding statement to attach any I-SID (new or inherited) to its own local UNI ports.

**SD-WAN/IPE confirmed NOT needed** for this two-switch CLI-driven lab POC — the entire tunnel/logical-interface/MTU/SPBM sequence above is native Fabric Engine/VOSS CLI; IPE only adds automated tunnel orchestration + app-aware WAN routing, neither required for a point-to-point lab test. Consistent with everything else established this session.

**Status: still concept/planning discussion — no hardware touched, no lab build started from this exchange.**

### POC architecture with a real "core": BCB/BEB roles + Karl's core + final topology (same session, still concept/planning)

**User's real goal (4 parts):** (1) IPEs to abstract WAN complexity for the client demo, (2) Fabric-over-EXOS/traditional coexistence at the edge — build the core once, never touch it, (3) demonstrate EP1 managing/visualizing the fabric, (4) a genuine core topology — "two BEBs separated over WAN is not good enough."

**Key KB finding: BCB vs BEB is a configuration role, not dedicated hardware or a license.** Fabric Engine 9.3 User Guide, verbatim: *"Configure all virtualization services on the BEBs at the edge of the network. There is no provisioning required on the core SPBM switches."* and *"A BEB performs the same functionality as a BCB, but it also terminates one or more VSNs. A BCB does not terminate any VSNs."* Any Fabric Engine-capable switch (5320, 5720, 7520, 7720, VSP) can be either role. "Building a core" = configuring SPBM/IS-IS with NNI-only interfaces and deliberately binding zero I-SIDs — not a separate procedure or SKU.

**Open KB gap, explicitly flagged (not resolved either way):** no explicit statement found on whether an FE tunnel can terminate directly on a bare BCB. Strongest signal found: the only documented I-SID-capable/service-gateway role is the BEB, and Extreme's own SD-WAN reference topology labels the hub-side FE endpoint **"BEB/BCB"** (combined role on one box), not a standalone pure-BCB. Working assumption: FE tunnel adjacency itself will come up fine on a bare BCB (IS-IS/SPBM + IP reachability is all FE needs), but the *service* (I-SID/VSN) has nowhere to terminate unless that same node also has UNI ports + an I-SID binding — i.e., functionally becomes a BEB/BCB combo, matching Extreme's own reference pattern.

**Finalized topology (Karl confirmed his core is a real, physical, single VOSS switch — assume one BCB, not virtual/EVE-NG):**

| Node | Status | Role |
|---|---|---|
| Core_BCB | Already built (Karl's) | Real VOSS, SPBM/IS-IS, NNI-only, zero I-SIDs |
| HQ_BEB | To build — needs a device Karl may or may not be able to provide | Local physical NNI to Core_BCB + UNI to APs/legacy EXOS + uplink to HQ_IPE |
| HQ_IPE | SD-WAN appliance | FE tunnel automation toward Remote |
| Remote_IPE | SD-WAN appliance | Mirror of HQ_IPE at MyLab/RS1 |
| Remote_BEB | To build (user's 2nd physical 5320) | FE tunnel back to HQ side, SPBM/IS-IS + I-SIDs, UNI to local APs |

Hardware math resolved: with Karl's Core_BCB covering the core role, the user's 2 physical 5320s were expected to cover HQ_BEB + Remote_BEB — no 3rd/4th physical unit or EVE-NG virtual node needed, *provided* a dedicated HQ_BEB device materializes.

**HQ_BEB ↔ Core_BCB local NNI is a genuine physical adjacency** — same case as the earlier Second_BEB exchange, so Zero Touch Fabric/Auto-sense applies here: HQ_BEB could boot near-vanilla and learn the B-VLAN from Core_BCB automatically rather than hand-matching it.

**Follow-up resolved: "can I skip a dedicated HQ_BEB and connect HQ_BCB directly to Remote_BEB via the IPEs?"** Answer: the FE tunnel/adjacency will work fine terminating directly on Core_BCB — that part doesn't require a separate BEB. But if Core_BCB stays a pure BCB (no UNI, no I-SID), Remote's services have no HQ-side termination point — usable only for Remote-to-Remote traffic (e.g. Remote_BEB ↔ a second remote BEB), not anything reaching HQ resources. Two ways to get real HQ-side service termination without new hardware: (1) ask Karl to add UNI ports + an I-SID binding directly onto his existing Core_BCB box, turning it into a combo BEB/BCB on the same hardware — this exactly matches Extreme's own SD-WAN reference "BEB/BCB" hub pattern; or (2) keep Core_BCB untouched and source a separate physical HQ_BEB device.

**Status: still concept/planning discussion — no hardware touched. Open action item: user to ask Karl whether (a) a dedicated HQ_BEB device is available, or (b) Karl is willing to add UNI/I-SID config directly onto his existing Core_BCB box.**

### SD-WAN/IPE fan-out + internet egress from a bare BCB (same session, still concept/planning)

**One hub IPE can serve multiple remote sites — confirmed.** KB: *"Automated Hub-and-Spoke Topology... automatic tunnels set up between spokes and hubs"* and *"a Hub ipe can route traffic between Spokes."* One HQ_IPE does not need to be replicated per remote site added later.

**Every site still needs its own local IPE — confirmed.** KB reference topology: *"a Hub (DC, with 2 ipes) and two Spokes (a Branch Site, with 1 ipe, and Headquarters, with 2)"* — reason given: *"Each ipe is paired with its own Fabric switch"* (1:1 per-site pairing with the local BEB, not a shared/centralized service). Unconfirmed/flagged: whether the hub's "2 ipes" in that example is a mandatory redundancy requirement or just Extreme's reference-design choice.

**Minimal 5-node POC confirmed as sufficient to test the model:** HQ_BEB, HQ_BCB (Karl's), HQ_SDWAN, Remote_SDWAN, Remote_BEB.

**Internet/external egress — confirmed structurally impossible from a bare BCB, reinforcing why a dedicated HQ_BEB matters.** KB: *"Core nodes act as pure BCBs that simply transport VSN traffic"* — no C-VLAN, no IP interface, no default gateway; a BCB cannot hand off to any non-fabric device, internet included. Two documented exit mechanisms, both requiring a BEB:
1. **IP Shortcuts** — *"the default router... uses IP shortcuts to IP route over the SPBM core"* — fabric acts as an L3 routed core, BEB is the default router for endpoints, no I-SID/VSN needed for that traffic.
2. **L3VSN** — an I-SID carries a routed IP subnet; a BEB terminates it and hands off routed traffic to non-fabric infrastructure.

Both mechanisms land the handoff at a BEB's UNI port to a non-SPBM device (firewall/router/ISP edge) — KB never associates this handoff with the BCB core. Concrete implication for the POC: internet access for either site must be homed off HQ_BEB (or Remote_BEB, if that site needs local breakout) via a UNI port to the existing router/firewall, using either IP Shortcuts or an L3VSN — never off Core_BCB directly. This is the same root cause as the earlier "HQ_BCB→Remote_BEB direct via IPEs" gap: fabric adjacency works fine on a bare BCB, but any real service or external handoff needs a BEB somewhere in the path.

**Status: still concept/planning discussion — no hardware touched.**

### MyLab 2-switch topology CLI + the VSN-to-internet mechanics gap, resolved (same session, still concept/planning)

**Scenario:** MyLab has 2 physical VOSS switches. Remote_BEB1 already connects to Karl's Core_BCB via local IPE/SD-WAN. Remote_BEB2 NNIs locally to BEB1, hangs its own APs (FA), and gets direct internet on Port-1.

**Remote_BEB1_IPE config — KB-confirmed pieces:**
- Auto-Sense on the port facing the IPE: `interface gig 1/x` then `auto-sense enable [convert-to-config]`; `show interfaces gigabitethernet auto-sense 1/x` should report state `SD-WAN` once the IPE is LLDP-detected.
- With Auto-Sense/SD-WAN engaged, the tunnel/logical-IS-IS-interface build is expected to be automatic (per the LLDP mechanism logged earlier this session).
- Manual fallback/reference sequence (same as logged previously): `ip-tunnel-source-address ... vrf fe` → `logical-intf isis <1-255> dest-ip <peer> mtu <val>` → `isis` / `isis spbm <id>` / `isis enable`, plus standard base SPBM/IS-IS bring-up.

**Remote_BEB2_Internet_Port1 — the VSN-to-internet hand-off gap, resolved into 3 distinct steps (not one mechanism):**
- **Step A — VSN termination (I-SID → C-VLAN), KB-quoted:** *"i-sid 20202 with c-vid 202 on port 1/5 and a VLAN interface with IP address 10.1.1.2/24"* — the I-SID decapsulates into a C-VLAN with its own ordinary VLAN IP interface, which is by default in the GRT. Once traffic is on that VLAN IP interface, it is no longer VSN/fabric traffic.
- **Step B — internet egress, KB-quoted concept (exact full CLI sequence not found, flagged as gap):** Port-1 as a brouter port (*"the slot and port number of the brouter port identifies brouter interfaces"*) with its own IP + a static default route (*"to configure a default static route, supply a value of 0 for the prefix and the prefix length"*), also in the GRT.
- **A→B connection — NOT an explicit single KB statement, this is my own synthesis connecting two independently confirmed facts:** since both the C-VLAN IP interface (Step A) and the brouter port (Step B) sit in the same GRT on the same switch, ordinary IP routing connects them with no extra config — KB: *"routed traffic in the GRT follows next-hop out of the routed interface context."* If only BEB2's own local clients need internet, Steps A+B alone are sufficient.
- **Step C — sharing that egress with other switches (e.g. BEB1), KB-confirmed exact CLI:**
```
interface loopback 1
ip address <loopback-IP>/32
router isis
ip-source-address <loopback-IP>
spbm <id> ip enable
redistribute direct
redistribute direct enable
redistribute direct metric 1
exit
isis apply redistribute direct
```
Validated by a direct best-practice quote: *"In cases where the Internet connection is single-homed, to reduce the size of the routing table, as a best practice, advertise Internet routes as the default route to the IGP"* — confirms `redistribute direct` (default route only) is the correct shape for Port-1's single-homed ISP link, not a full route dump into the fabric.

**Net resolution of the user's stated gap ("how the BEB terminates the VSN and hands over to the internet, I cannot [architect]"):** there is no single "hand-off" mechanism — it's two independent, ordinary GRT-routing steps (I-SID→C-VLAN, and brouter-port→default-route) that happen to coexist on the same switch and therefore route to each other for free; IS-IS/IP-Shortcuts redistribution is a separate, optional third step, needed only when a different switch must share that same egress.

**Status: still concept/planning discussion — no hardware touched.**

**User's business rationale, confirmed/clarified (same session):** the case for VOSS over EXOS in the client-migration story isn't "less core work during buildout" — it's ongoing operational burden post-deployment for a small IT staff. Once VOSS/Fabric is stood up, edge changes (new AP, new I-SID, new VLAN) never require touching the core (edge-only provisioning, confirmed earlier this session). EXOS/VLAN sprawl requires hop-by-hop touching of every intermediate switch on every edge change. User's framing: "EXOS is more work after and at deployment for a small IT_Staff to manage. VOSS, once up, is superior." Worth reusing as a concrete selling point (low ongoing-maintenance burden for small IT teams) in the hospital hub-spoke client scenario logged earlier this session.

### IQController vs IQEngine terminology + fact-check of a pasted 3rd-party dialogue on PPSK/UZTNA I-SID mapping for single-SSID medical IoT (new session, Sept 2)

**Terminology, KB-confirmed:**
- **IQEngine** = the AP operating system/software persona running on the access point itself (formerly HiveOS) — not a management platform.
- **IQController / ExtremeCloud IQ Controller / XIQ-C** = a separate, centralized **controller** platform for on-prem campus deployments (airgap licensing, tunneling, distributed/centralized data planes, integrates with Site Engine/ExtremeControl/ExtremeAnalytics/AirDefense) — a different product line from cloud-managed EP1/XIQ entirely.
These are not two names for the same thing and not interchangeable — one is AP firmware, the other is an on-prem controller product.

**Fact-check of user's pasted dialogue (a different AI's multi-turn answer on hospital single-SSID medical-IoT segmentation via PPSK):** that transcript made three successive attempts at describing an EP1 UI workflow (VLAN Profile → Topology Type "Fabric Attach" → I-SID field → User Profile → PPSK User Group → SSID), with the user confirming two of the three attempts were wrong ("This does not exist," "There is no provision in EP1 for this"). KB re-check of the third/final claim, this session:
- **CONFIRMED real, KB-quoted:** *"The Fabric Attach topology type is similar to B@AP with the added I-SID parameter"* and *"I-SID — For Fabric Attach. A unique VLAN identifier and a unique I-SID (service identifier). The I-SID range is (0-15999999)."* So a VLAN Profile with Topology Type "Fabric Attach" exposing an I-SID field is real and documented (ExtremeCloud IQ Controller v10.12.01 User Guide; also present in EP1/XIQ-New's device VLAN+I-SID dropdown workflow).
- **NOT confirmed — the missing link:** tying that I-SID/VLAN-Profile mechanism directly to a **PPSK User Group** on an SSID. KB confirms PPSK user groups exist and are configured at the SSID level (Configure → User Management → User Groups), and confirms different user profiles/PPSKs can be assigned per-client on the same SSID — but no KB excerpt shows an end-to-end "PPSK User Group → I-SID" binding procedure. VLAN/topology assignment in the KB is described as flowing through a **configuration Profile** (network + role → Profile → VLAN), not directly through the PPSK group object. Net: the I-SID field itself is real; the specific PPSK-group-to-I-SID wiring claimed in the pasted transcript is unverified and should not be treated as confirmed.

**Open, not yet addressed:** the user's own live/concrete issue — described only as "an issue with IQEngine with the VLAN-i-SID mapping" — has no specifics captured yet (no error text, no `show fa assignment`/EP1 screenshot, no switch/AP identified). Needs the user's actual symptom before this can be diagnosed.

**Status: concept/fact-check discussion — no hardware touched yet this sub-thread.**

### LIVE HARDWARE SESSION, Sept 2 — KhKLab-SW-01 FA/FE/Auto-Sense diagnosis + working 3-in-1 demo capture

**Topology confirmed live:** `KhKLab-SW-01` (SW1) — port 1/1 → SD-WAN appliance → RDU-Core (`FishBowl-Fabric_Core-5720-01`, sysID e0a1.29fc.4084); port 1/10 → direct NNI → SW2 (`5320-16P-2MXT-2X-FabricEngine`, sysID d8e0.1637.6c84); ports 1/3 and 1/7 → AP3000s.

**Fabric Attach qualification, resolved against KB:** neither the SD-WAN/FE link (port 1/1) nor the SW1↔SW2 NNI (port 1/10) is Fabric Attach — both are genuine Fabric NNI / SPBM IS-IS adjacencies between fabric-native devices. KB: *"Fabric NNI (Link between fabric devices)"* vs *"Fabric Attach NNI (Link to Fabric Attach)"*; FA is specifically for non-SPB-capable edge devices over a Switched UNI. FA only qualifies on 1/3 and 1/7 (the AP-facing ports).

**Root-cause diagnosis walked live, AP3000 not getting proper FA mapping:**
1. `show fa elements` initially showed STATE `T / D` (Tagged / **AutoConfig Disabled**) on both 1/3 and 1/7, despite successful discovery + auth (`successAuth`/`successAuth`). KB: STATE legend = Tagging/AutoConfig; `D` = AutoConfig disabled per-port, independent of the physical port itself.
2. Fix applied: `fa port-enable 1/7` and `fa port-enable 1/3`. Confirmed via `show fa interface port X`: SERVER STATUS flipped to `enabled`. (MGMT ISID/MGMT CVID stayed `0/0` on that command — that field is FA's own management-plane binding, separate from the per-client dynamic mapping; not a fault indicator by itself.)
3. Real confirmation came from `show fa assignment`: both ports show `active`/`client`-origin I-SID assignments — FA is mechanically working end-to-end (discovery → auth → auto-provision).

**Unresolved real issue found, still open:** `show fa assignment` shows **1/3 → I-SID 143154** (VLAN 154, generic name `ISID-143154`) vs **1/7 → I-SID 144154** (VLAN 154, named `BOBKit-4-WirelessUser-isid`). Same VLAN tag, two different I-SIDs — since I-SID (not VLAN) is the actual fabric isolation boundary, these two APs are landing in two separate, non-communicating VSNs despite sharing a VLAN number. Likely cause (inference, not KB-confirmed): the two APs are bound to two different EP1 Network Policies/VLAN Profiles, each independently specifying its own Fabric-Attach I-SID for "VLAN 154" instead of a shared value. **Still waiting on user to confirm intent** (shared single VSN for roaming vs. deliberate per-AP isolation) before prescribing the EP1-side fix.

**Full 3-mechanism demo, captured live and working simultaneously:**
- `show interfaces gigabitethernet auto-sense 1/1-1/10` → `1/1 = SD-WAN`, `1/10 = NNI-ISIS-UP`, `1/3 = FA`, `1/7 = FA` (matches KB-documented state names exactly, though this build reports generic `FA` rather than the `FA-WAP` sub-type shown in some doc examples).
- `show isis adjacencies` → two simultaneous ACTIVE Level-1 adjacencies: `SD-WAN-1` to FishBowl-Fabric_Core-5720-01 (Fabric Extend, uptime 05:46:06) and `Port1/10` to SW2 (plain NNI, uptime 00:55:16). Best single screenshot for "FE and local NNI are both just IS-IS adjacencies to the fabric."
- `show isis lsdb` → 14 LSP entries across **5 systems** in the HOME area: TSLab-SW-01, SW2, KhKLab-SW-01, FishBowl-Fabric_Core-5720-01, Fishbowl-SW-01 — proves full fabric-wide LSDB visibility beyond just the two direct neighbors (SPB flooding reaching non-adjacent nodes).
- `show isis spbm` SMLT info shows `SMLT-SPLIT-BEB: primary`, all-zero virtual B-MAC, no peer system ID — default/inactive SMLT state (no peer configured), unrelated to the FE/FA/Auto-Sense demo, not a concern.

**New issue opened at session end, NOT yet diagnosed:** user reports the AP plugged into port 1/7 is now showing **down** (physical/link state, not the FA mapping issue above — that AP was FA-active moments earlier in this same session). No diagnostic commands run yet on this. Next step: get `show interfaces gigabitethernet 1/7` (or equivalent port/link-state command) and current `show fa elements`/`show fa interface port 1/7` to see whether this is a physical link-down, a PoE issue, or FA state regression.

**Status: live hardware session, actively in progress — AP 1/7 link-down issue is the open thread.**
