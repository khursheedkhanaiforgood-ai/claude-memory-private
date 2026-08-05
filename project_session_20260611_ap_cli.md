---
name: AP CLI + Networking Session — June 11-12 2026
description: IQ Engine CLI learnings, EP1 reverse tunnel SSH, VoIP basic rate/stickiness, stationary call drops, home lab VPN options. For EOD HTML.
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
# AP CLI Session — June 11 2026

## Key Learnings (for EOD HTML)

### Pagination
- `terminal length` = Cisco IOS syntax — INVALID on IQ Engine (returns unknown keyword)
- IQ Engine alternative: `show log buffered 100` (last N lines) or `| more` pipe
- Practical workaround: terminal emulator session logging + scroll buffer

### _debug vs _kdebug
- `_debug` = userspace daemon debugging (auth, dhcp, roaming, amrp processes)
- `_kdebug` = kernel/driver layer (802.11 frame events, key handshakes, association state machine)
- Output streams live to console by default — not silently buffered in background
- Redirect to file: `_debug auth all logfile /tmp/auth_debug.log`
- Read file: `show file /tmp/auth_debug.log`
- Stop: `no _debug all` / `no _kdebug wifi0 all`
- `/tmp` = RAM only — lost on reboot
- Detail doesn't exist until flags are turned on (surgeon dictation analogy)
- Troubleshooting sequence: clean slate → start debug → trigger event → stop → `show log buffered 200`
- `_kdebug wifi0 all` can flood console on busy AP — use `_kdebug wifi0 mgmt` for scoped capture

### VLAN on client from AP
- Command: `show station` — shows all associated clients including VLAN column
- Per-client detail: `show station <mac-address>`

### show station — VLAN + UPID mapping
- `show running-config | i vlan` → `| i` pipe = IQ Engine's grep (case-insensitive include)
- user-profile attribute number = UPID column in `show station`
- WC_TeamUSA VLAN 10 / UPID 53, WC_TeamAUS VLAN 20 / UPID 54, WC_Seattle_Guest VLAN 30 / UPID 50

### EP1 Reverse Tunnel SSH
- Direct SSH to 192.168.0.12 fails from remote network (private IP, no route)
- EP1 allocates ephemeral public port (e.g. 3.145.235.126:20197) bridged back through IQAgent tunnel
- AP makes outbound connection to cloud — no inbound port needed, works through any NAT/firewall
- Port is temporary — closes when EP1 session ends
- From home network (same subnet as AP): direct SSH always works — no tunnel needed

### VoIP Basic Rate / Stickiness (customer issue June 12)
- Beacons sent at lowest basic rate → client stays connected as long as it decodes beacons
- 6 Mbps basic → beacon threshold ~−82 dBm → very sticky, call drops at −80 dBm (13 dB below VoIP floor)
- 24 Mbps basic → beacon threshold ~−74 dBm → less sticky, client roams sooner
- Coverage shrinkage: 10·log(new/old) dB per doubling = 3 dB; ~21% less range indoors (n=3)
- Customer issue: stationary call drops → NOT a roaming/stickiness problem
- Stationary drop suspects: marginal RSSI, high CU/retries, WMM misconfigured, GTK rotation (periodic drops), DTIM/power save mismatch
- Key diagnostic: "Are drops on a regular rhythm (GTK/DTIM) or random (interference/SNR)?"

### Home Lab VPN — Process (for EOD HTML)

#### Option A — Tailscale (recommended: zero config, 15 min)
1. Install Tailscale on always-on home machine (Mac Mini, Pi, NAS): `brew install tailscale` or pkg
2. `tailscale up` → authenticate with Google/GitHub account
3. Install Tailscale on travel laptop → same account
4. Tailscale assigns each device a stable 100.x.x.x IP
5. From hotel: `ssh admin@192.168.0.12` still won't work — but `ssh user@100.x.x.x` (home machine) then SSH to AP from there works
6. Better: enable **subnet routing** on home machine → `tailscale up --advertise-routes=192.168.0.0/24`
7. Enable route in Tailscale admin console
8. From hotel: `ssh admin@192.168.0.12` works directly — Tailscale routes it through home machine
9. No port forwarding on QF_Modem required — Tailscale uses NAT traversal (same principle as EP1 tunnel)

#### Option B — WireGuard on home router (if QF_Modem supports it)
1. Check router admin → VPN section → WireGuard or OpenVPN server
2. Many Asus, GL.iNet, pfSense, OPNsense routers have native WireGuard server
3. Generate server keypair, set tunnel subnet (e.g. 10.99.0.0/24), set listen port (e.g. 51820)
4. Forward UDP 51820 on QF_Modem to router WAN IP
5. Install WireGuard client on laptop → import config → Connect
6. Full tunnel: all traffic routes home including 192.168.0.x access
7. Requires: port forwarding on QF_Modem + static/DDNS home IP

#### Option C — Tailscale SSH (simplest for AP access only)
- Install Tailscale directly on a Raspberry Pi on home network
- Pi joins Tailscale mesh → accessible at 100.x.x.x from anywhere
- SSH to Pi first, then `ssh admin@192.168.0.12` from Pi
- No subnet routing needed, no port forwarding, Pi can be always-on

#### Recommendation for this lab
Tailscale + subnet routing (Option A) = 15 min setup, works from any network, no port forwarding, 
same NAT-traversal principle as EP1. Once set up, `ssh admin@192.168.0.12` works from hotel 
exactly as it does at home.

#### CONFIRMED PLAN — Raspberry Pi 4 + Tailscale (June 12 2026)
User buying hardware today. No programming — copy-paste CLI only. ~20 min total setup.

**Shopping List (save to EOD HTML):**
- Raspberry Pi 4 (2GB RAM) — ~$45 (CanaKit or official)
- MicroSD card 16GB+ (Class 10 / A1 rated) — ~$8 (SanDisk recommended)
- USB-C power supply (5V/3A official Pi supply) — ~$10
- Short ethernet cable (Cat5e/Cat6) — likely already have one
- Total: ~$63

**Where to buy same day:**
- Micro Center (if local) — best Pi prices in-store
- Best Buy / Canada Computers — carries CanaKit bundles (~$80 all-in)
- Amazon same-day if Prime available

**Full Setup Guide (for EOD HTML):**

Step 1 — Flash SD card (10 min)
- Download Raspberry Pi Imager: raspberrypi.com/software
- Insert SD card into Mac
- Select: Raspberry Pi OS Lite (64-bit) — no desktop needed
- Click gear icon → enable SSH → set username/password → set hostname: lab-pi
- Write → eject

Step 2 — First boot (5 min)
- Insert SD into Pi, plug ethernet into QF_Modem, plug USB-C power
- Wait 60s for boot
- From MacBook: `ssh pi@lab-pi.local` (or find IP in QF_Modem DHCP table)
- `sudo apt update && sudo apt upgrade -y`

Step 3 — Install Tailscale (2 min)
- `curl -fsSL https://tailscale.com/install.sh | sh`
- `sudo tailscale up --advertise-routes=192.168.0.0/24 --accept-routes`
- Click auth URL → sign in with Google account

Step 4 — Approve subnet route (1 min)
- Go to login.tailscale.com → Machines → lab-pi
- Click "..." → Edit route settings → enable 192.168.0.0/24
- Disable key expiry on lab-pi (so it stays connected forever without re-auth)

Step 5 — Laptop setup (2 min)
- Download Tailscale app on MacBook: tailscale.com/download
- Sign in → same Google account
- Toggle on

Step 6 — Test from any network
- `ssh admin@192.168.0.12` → works from hotel, coffee shop, anywhere
- `ping 192.168.0.12` → confirm routing

**Ongoing maintenance:** None. Pi runs headless 24/7 on ~4W.
**Reboot Pi remotely:** `sudo reboot` over SSH or Tailscale SSH.
**Pi IP stays stable:** Set DHCP reservation in QF_Modem for Pi's MAC address.

### AP1 Running Config + Show Tech Review (June 15 2026)

**Show Tech Key Findings:**
- 3 reboots June 10 (15:41, 15:59, 16:11) + multiple May 20-21 — root cause: config rollback enabled + EP1 policy pushes
- RadSec SSL timeouts recurring (May 20, Jun 9, Jun 10, Jun 15 overnight) — PPSK auth fails when cloud TLS session drops
- Walled-garden DNS slow — Guest CWP portal delays
- DHCP lease = 3600s (1hr) — too short, AP loses mgmt IP on renewal failure → reboots
- Location-essentials enabled → hundreds of Locn-essential CAPWAP events per minute (normal but load contributor)
- Console login failures June 10 = our session (backspace escape chars in log)

**Running Config Key Findings:**
- Basic rate = 6 Mbps on both SSIDs → sticky clients, bad for VoIP → raise to 12 Mbps
- CRC threshold = 35% but saw 47% → ACSP should have triggered, verify EP1 not overriding
- Config rollback enabled → EP1 push + CAPWAP disconnect = reboot cycle
- ADSP not in running config → pushed from EP1 cloud policy (explains why show running-config | i adsp returns nothing)
- Default fallback UPID = attr 7 → VLAN 30 (safe, not untagged VLAN 1)
- SNR steering active on PPSK SSID (disassoc + BSS transition request)

**Roaming Cache Review:**
- Roaming: enabled (802.11r active, confirmed June 10)
- Max Caching Time: 3600s (1hr) — should match DHCP lease, both should be 86400s (24hr)
- Roaming hops: 1 — fine for single AP lab; increase to 2-3 for multi-AP deployments
- Roaming hops = how far PMK cache propagates to neighboring APs
  - 1 hop = direct neighbors only
  - 2-3 hops = cache pre-propagates further so client roam destination already has PMK
  - At 1 hop, client roaming beyond direct neighbor = full re-auth = 200-500ms = VoIP drop risk
- Cache empty at time of capture (no clients associated)

**Priority Actions (to execute in EP1):**
1. Disable ADSP on radio profile → enables pcap capture
2. Raise basic rate to 12 Mbps on both SSIDs
3. Increase PMK max caching time to 86400s (24hr) — align with DHCP lease
4. Set roaming hops to 2 (CONFIRMED June 15) — covers fast walkers; 3 only for carts/forklifts
5. Fix DHCP lease to 24hr on QF_Modem (static reservation for 192.168.0.12)

**Roaming Hops + Client Speed (June 15):**
- Roaming hops = how far PMK cache pre-propagates to neighboring APs
- Indirectly tied to client mobility speed: faster clients traverse AP boundaries quicker
- At 1 hop: client skipping a neighbor (fast walk) hits an AP without PMK → full re-auth → 200-500ms gap → VoIP drop
- At 2 hops: cache is pre-positioned one level further → fast walkers covered
- At 3 hops: warranted for high-mobility (carts, forklifts, jogging)
- For office walking pace: 2 hops is correct call
- CONFIRMED: this lab set to 2 hops

**Three-way alignment required (all must match):**
- DHCP lease time (QF_Modem) → 86400s (24hr)
- PMK max caching time (EP1 roaming settings) → 86400s (24hr)
- Roaming cache max caching time (AP roaming profile) → 86400s (24hr)
- All three currently at 3600s (1hr) — misalignment causes CAPWAP drop → reboot cycle

### AP2 Onboarding — June 16 2026 (AH-565780)

**Full sequence:**
- Cable reseat on SW1 port 5 resolved PoE (was 1.1W detection only; `show inline-power info ports 5` = correct syntax)
- Console login: admin / aerohive (post factory reset default)
- `reset config` = IQ Engine factory reset (clears VIQ CAPWAP target)
- CAPWAP states: DISCOVERY → SULKING (900s pause after 3 failed attempts) → RUN (DTLS connected)
- Wrong serial claimed first (AH-901340). Correct serial = AH-565780 from `show version` at console prompt
- After correct claim in EP1 + second `reset config`: DISCOVERY → RUN in <60s
- AP2 rebooted ~3 min after first connect — EP1 auto-pushed firmware upgrade (normal on first claim)
- Port 5 VLAN tagging required manually; EP1 Supplemental CLI updated from ports 3 → ports 3,5

**SW1 DHCP + ipforwarding:**
- `enable dhcp-server` = INVALID on EXOS. Correct: `enable dhcp ports 3,5 vlan <name>` per VLAN
- `enable ipforwarding` = needed for inter-VLAN routing + internet (run manually, NOT in Supplemental CLI)
- Both NOT idempotent — stall EP1 push at 15% on re-run
- SSH hostkey blocks after AP factory reset: `ssh-keygen -R <ip>` to clear then reconnect

**EP1 Stale Cache + MAC Randomization — June 16:**
- EP1 showed MacBook associated to AP1; `show station` on AP1 → no clients; `show station` on AP2 → no clients
- MacBook had IP 10.30.0.102 on correct SSID — definitely connected somewhere
- **Dual root cause:** (1) EP1 stale cache — AP2 came online, client may have roamed, EP1 hadn't refreshed. (2) macOS MAC randomization — macOS rotates MAC per SSID on reconnect. Cached MAC no longer current.
- **Rule: `show station` on the AP is always authoritative. EP1 client view is cached.**
- Diagnostic chain: show station ALL APs → `networksetup -getmacaddress en0` on MacBook → `show fdb ports 3,5` SW1 → `show iparp vlan Guest` SW1
- Client association debug ONGOING — turn WiFi off/on on MacBook, check show station immediately

**XE Scenarios — Karl Method — June 15:**
- 8 scenarios completed: XE-11 (6GHz RNR/WPA3), XE-12 (DFS TDWR 5.6GHz), XE-15 (CU/CRC RF), XE-16 (DFS radar), XE-17 (3GHz/load balance), XE-19 (NAC quarantine), XE-20 (PCG L2-only isolation), XE-21 (PPSK bulk expiry)
- Total done: 9 (XE-08 prior + 8 today). 41 remaining.
- Grade: B+ / 8.2 out of 10
- Karl Method: brief situation → user asks questions → terse answers only → no hints volunteered

**EOD HTML:**
- June 15: `docs/session_summary_20260615_ap_cli.html` — commits 0ef1202, 141cf79, aa9fb58, 2c8634a
- June 16: `docs/session_summary_20260616_ap2_onboard.html` — commit 51b6ce7
- All 91 HTML pages: blinking red warning banner (June 15 2026 date, VERIFY WITH SME FIRST)
- Banner CSS fix: `width:100%;box-sizing:border-box;display:block` (flex body collapses banner otherwise)

**Why:** User learning AP CLI + CAPWAP onboarding fundamentals in parallel with DigitalTwinEngine pcap sprint.
**How to apply:** Reference for June 15-16 EOD HTML artifacts + Socratic CLI scenario bank context.
