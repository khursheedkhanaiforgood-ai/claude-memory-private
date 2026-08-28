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
