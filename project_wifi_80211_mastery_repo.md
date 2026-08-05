---
name: wifi-80211-mastery Repo — Full EOD State May 25 2026
description: NEW PUBLIC GITHUB REPO created May 25 2026. 802.11 deep-dive curriculum — 6 phases, 34 days, 11 HTML files. Includes CWNA-108 + 802.11be + EP1 cross-refs and 46-Q phase assessments + 30-Q CWNA mock. Live on GitHub Pages. Verbatim for EOD HTML.
type: project
originSessionId: 52c71221-9cc9-44db-9c54-689b1e8f03fe
---
## REPO IDENTITY

- **Name**: `wifi-80211-mastery`
- **GitHub**: https://github.com/khursheedkhanaiforgood-ai/wifi-80211-mastery
- **GitHub Pages (live)**: https://khursheedkhanaiforgood-ai.github.io/wifi-80211-mastery/
- **Visibility**: Public
- **Created**: May 25 2026 (this session)
- **Pages source**: `main` branch, root path
- **Local clone**: `/tmp/wifi-80211-mastery/` (private worktree, will be cleaned up)
- **Description**: "802.11 deep-dive curriculum — Socratic study schedule, NYT-style HTML knowledge base, lab exercises on AP3000 + 5320 EXOS"
- **Total commits**: 3 (1f27401 scaffold → 52053f4 references → fc4b725 assessments)

## ALL FILES (11 HTML)

| File | Purpose | Color |
|------|---------|-------|
| `index.html` | Landing page — 7-card grid (6 phases + References + Assessments) | mixed |
| `docs/study_80211_index.html` | Hub — day-by-day list of all 34 study days | mixed |
| `docs/study_phase1_phy.html` | Phase 1 — 8 days PHY/RF | teal `#134e4a` |
| `docs/study_phase2_mac.html` | Phase 2 — 6 days MAC | orange `#7c2d12` |
| `docs/study_phase3_security.html` | Phase 3 — 6 days Security | rose `#9d174d` |
| `docs/study_phase4_qos_roaming.html` | Phase 4 — 5 days QoS/Roaming | violet `#581c87` |
| `docs/study_phase5_capacity.html` | Phase 5 — 5 days Capacity | blue `#1d4ed8` |
| `docs/study_phase6_capstone.html` | Phase 6 — 4 days Capstone | emerald `#065f46` |
| `docs/references.html` | CWNA-108 + 802.11be + EP1 cross-map | black `#1a1a1a` |
| `docs/assessments.html` | 6 phase quizzes + CWNA-108 mock | amber `#b45309` |

## CURRICULUM STRUCTURE (34 days, ~55–60 h)

### Phase 1 — PHY & RF Internals (8 days, ~12 h, teal)
1. OFDM symbol structure & subcarrier spacing across 11a/n/ac/ax/be
2. MCS index mechanics — SNR floor, cell-edge collapse
3. Channel widths 20/40/80/160/320 MHz — primary/secondary
4. 5 GHz DFS & FCC channels — radar cascade
5. 6 GHz / Wi-Fi 6E — PSC, AFC, RNR/FILS/UPR
6. MU-MIMO vs OFDMA — when each wins
7. Spatial streams, TxBF sounding (NDP/NDPA)
8. Wi-Fi 7 PHY — 320 MHz, 4K-QAM, MLO link types, multi-RU

### Phase 2 — MAC & Frame Exchange (6 days, ~9 h, orange)
1. DCF, EDCA AC queues, CWmin/CWmax/AIFSN per AC
2. NAV, RTS/CTS, MU-RTS, BSS Color spatial reuse
3. Aggregation — A-MPDU vs A-MSDU, BlockAck windows
4. TWT — individual, broadcast, restricted
5. Power save — PS-Poll, U-APSD, OMI, TIM/DTIM
6. Multicast/broadcast — DTIM cost, ARP/ND, IGMP snoop

### Phase 3 — Security Beyond the Handshake (6 days, ~10 h, rose)
1. PMKSA caching, OKC, FT (802.11r) keying hierarchy
2. SAE H2E (Hash-to-Element), PT, anti-clogging
3. MFP / 802.11w / BIP — protected mgmt frames
4. OWE + OWE Transition Mode
5. 802.1X EAP-TLS — cert chain, MAC randomization vs RADIUS
6. Roaming triggers — 11k Neighbor Report, BTM (11v), Apple algo

### Phase 4 — QoS, Roaming & Convergence (5 days, ~8 h, violet)
1. DSCP↔UP mapping, WMM-UP↔AC, DSCP=0 collapse problem
2. Uplink QoS trust on AP3000, downlink AC selection
3. Voice roaming budget — 50 ms target, 11r vs OKC cost
4. Sticky clients & RSSI hysteresis, controller vs client
5. Load balancing — band steering, client distribution, 11v MBO

### Phase 5 — Capacity Planning & HDD (5 days, ~9 h, blue)
1. Airtime math — ToA per MCS × frame size + overheads
2. RF coverage tiers — −67/−72/−75 dBm, RSSI asymmetry
3. SSID overhead — beacon airtime cost per SSID per band
4. Co-channel vs adjacent-channel interference
5. Wi-Fi 7 capacity model — MLO + multi-RU + 320 MHz ceiling

### Phase 6 — Field Troubleshooting Capstone (4 days, ~7 h, emerald — terminal)
1. WLPC 2026 6-process framework (MANAGE/INVESTIGATE/MEASURE/VALIDATE/ANALYZE/INTERROGATE)
2. 30-row causes table — WIRELESS / LOCAL NET / INTERNET
3. Frame stats — retries, BAR, retry % thresholds
4. Capstone — build WLPC-style incident report (output: `docs/study_capstone_report.html`)

## REFERENCES PAGE — 4 SECTIONS (`docs/references.html`)

### A. CWNA-108 Chapter Map (34 rows)
Every curriculum day mapped to CWNA-108 chapter topic + note on where curriculum goes deeper. CWNA-109 was claimed by Claude but user couldn't confirm publication → reverted to CWNA-108 (Sybex, Westcott & Coleman, 2020) by user decision.

### B. 802.11be / Wi-Fi 7 Deep References (15 features)
- EHT PPDU formats (§35.3) → P1.8
- 320 MHz channel — 6 GHz only (§27 + §35.5) → P1.3, P1.5
- 4K-QAM MCS 12/13 (§35.7) → P1.2, P1.8
- MLO core (§35.3.6 + §11.55) → P1.8, P5.5
- MLO STR (§35.3.6.2) → P1.8, P4.3
- MLO NSTR (§35.3.6.3) → P1.8
- EMLSR (§35.3.6.4) → P1.8, P4.3
- EMLMR (§35.3.6.5) → P1.8
- Multi-RU (§35.5.4) → P1.6, P1.8
- Preamble puncturing (§35.3.4) → P1.3
- Triggered Uplink Access (§35.6) → P2.1, P2.2
- BSS Color enhancements (§35.10) → P2.2
- R-TWT (§35.6 ext) → P2.4
- EHT security SAE-PT, PASN (§12 + §35) → P3.2
- EHT capacity gain analysis (§Annex Z) → P5.5

### C. EP1 Cross-Links to 5320-onboarding Repo
Branch: `feature/xiq-ep1-intelligence-engine` (will be merged to main).
- P3.1 FT/OKC → `ep1-deployment-guide.html`
- P3.5 802.1X → `ep1-deployment-guide.html`
- P3.6 11k/v BTM → `ep1-deployment-guide.html`
- P4.1–4.2 DSCP/QoS → `xiq-ep1-arch-debate.html`
- P4.4–4.5 Sticky/steering → `xiq-ep1-transition.html`
- P5.1–5.3 Capacity → `xiq-ep1-intelligence-engine.html`
- P6.1–6.3 WLPC → `xiq-ep1-design-dialogue.html`
- P3.3 MFP/rogue → `ge-airdefense-xiq-sprint.html`
- Cross-cutting API/SNMP → `ep1-api-snmp-study.html`

### D. WLPC Corpus Tier Reminder
- T1 Authoritative: 80211-2024.pdf, 802.11be-2024.pdf, WiFi-6-For-Dummies (Extreme), Rev Wi-Fi Planner, WLANPros Checklists v5, WLPC Express 2026
- T2 Reference: Aruba VHD set, Apple Wireless Documents
- T3 Tools: WLAN Pi, Wireshark profiles, sample PCAPs
- **Privacy rule: citations only — never embed/mirror/link the PDFs from public pages**

## ASSESSMENTS PAGE — 7 QUIZZES (`docs/assessments.html`)

### Quiz 1 — Phase 1 PHY (10 Qs, ~40 min)
Q1 Calc: 11ax symbol time given 78.125 kHz spacing → 13.6 μs (12.8 μs data + 0.8 μs CP)
Q2 SNR floor match: MCS 0/4/7/9/11 → 2/18/25/30/35 dB
Q3 Scenario: Channel 36 primary vs ch 40 primary on 80 MHz — co-channel cooperative
Q4 DFS: Radar on secondary ch 56 forces entire 80 MHz block off; 30-min non-occupancy
Q5 6 GHz PSC: Non-PSC AP placement cost; RNR IE in 5 GHz beacons + FILS/UPR recovery
Q6 MU-MIMO vs OFDMA: 30 voice clients → OFDMA wins (small frames, many clients)
Q7 TxBF: 2 SS iPhone benefits from 4 AP antennas (~3–6 dB aperture gain)
Q8 Wi-Fi 7 MLO STR→EMLSR fallback: insufficient frequency separation; STR=throughput+latency, EMLSR=latency only
Q9 Drawing: subcarrier spacing vs symbol time vs CP overhead
Q10 Synthesis: −68 dBm at MCS 4 is suspicious — check retry % and noise floor

### Quiz 2 — Phase 2 MAC (8 Qs, ~30 min)
Q1 EDCA tie-break: AC_VO wins by design (smaller CWmin)
Q2 BSS Color OBSS_PD: heard −74 dBm, threshold −72 dBm → transmit; IE = Spatial Reuse Parameter Set
Q3 A-MPDU vs A-MSDU in lossy RF: A-MPDU (per-subframe retry granularity)
Q4 TWT calc: 60s sleep × 100 ms wake → 6 sec/hour = 0.17% airtime
Q5 TIM/DTIM: 200 ms wake spike = large multicast burst at DTIM
Q6 IGMP snoop: doesn't save WiFi airtime; need AP MC2UC conversion
Q7 RTS Duration/ID 800 μs = NAV reservation
Q8 U-APSD vs TWT: TWT lower drain for predictable; U-APSD flexible for sporadic

### Quiz 3 — Phase 3 Security (8 Qs, ~35 min)
Q1 FT key hierarchy: AP1 holds PMK-R0; PMK-R1 derived per-AP via DS
Q2 SAE H2P vs H2E: H2P had timing side-channel; H2E uses PT pre-comp; IE = RSN Extended Cap
Q3 MFP rogue Deauth: SA Query Request → Response
Q4 OWE-T: OWE Transition Mode IE in open beacon points at OWE BSSID
Q5 EAP-TLS missing intermediate CA: fails at Server Hello+Cert, TLS Alert bad_cert/unknown_ca
Q6 MAC randomization breaks authorization (Calling-Station-Id Attr 31)
Q7 Apple BTM behavior: needs ~8 dB hysteresis margin
Q8 OKC vs FT latency: FT eliminates post-assoc EAPOL exchange

### Quiz 4 — Phase 4 QoS/Roaming (7 Qs, ~30 min)
Q1 DSCP=0 collapse: iOS marks WMM UP at air interface based on app identity
Q2 AP3000 uplink trust: enforced at AP wired egress (remarking/rate-limit)
Q3 Voice roam budget calc: 150−70−30=50 ms budget; 100 voice frames lost if >50 ms
Q4 Sticky iPhone at −78 vs AP2 −63: likely SSID/security mismatch
Q5 Band steering: probe suppression breaks iPhone scan; use 11v MBO Status Code 30
Q6 DSCP 34 (AF41) → UP 5 (AC_VI); RFC 8325 updates mapping
Q7 Hospital VoIP roaming: enable 11r + 11k + 11v, 6–8 dB hysteresis, min-RSSI −72 dBm

### Quiz 5 — Phase 5 Capacity (7 Qs, ~30 min)
Q1 Airtime calc: 30 × 2 Mbps / 52 Mbps MCS 5 = ~115% → over-subscribed
Q2 RSSI asymmetry: uplink is constraint (AP has more antennas)
Q3 SSID overhead: 4 SSIDs × ~500 μs ToA × 10/sec × 2 bands = ~4%
Q4 Co-channel vs adjacent at 5m: co-channel wins (cooperative CCA)
Q5 Wi-Fi 7 ceiling: 30 clients × 5 Mbps = 150 Mbps; capacity not constraint, EDCA contention is
Q6 Reduce overhead: raise mandatory rate to 24 Mbps (cuts broadcast by 75%)
Q7 KhKLab design synthesis: −67 dBm, 24 Mbps mandatory, 3 SSIDs, 15–20 voice clients

### Quiz 6 — Phase 6 Capstone (6 Qs, ~25 min)
Q1 May 7 DHCP framework mapping: customer report=MANAGE, empty ARP=MEASURE, before/after PCAP=INTERROGATE, en8 repro=VALIDATE
Q2 DHCP NAK 4 rows: VLAN access, AP mgmt VLAN, WPA2 key state desync, 4-way handshake failure
Q3 18% retry vs 4% cell: (1) client RF, (2) interference, (3) driver/firmware bug
Q4 Incident report skeleton: 6 sections matching 6 WLPC processes
Q5 WLANPros v5 DTIM/multicast test
Q6 Single most valuable lesson: **Capture before reboot**

### CWNA-108 Mock — 30 Multiple Choice (~50 min, 70% pass = 21/30)
30 questions across: RF Fundamentals (1-2), 802.11 Standards (3, 17, 29), MAC (4-5), WLAN Arch (6), Security (7-12), Site Survey (13-14, 23-24), DFS (15), PoE (16), 802.11ax (18-21), Attacks (22), SNR (25), BYOD (26), Roaming 11k (27), Troubleshooting (28), Vertical Markets (30).

All answers in `<details>` collapse blocks for self-scoring.

## COMMITS (in order)

1. `1f27401` (May 25) — Phase 0 scaffold: 802.11 deep-dive curriculum (6 phases, 34 days) — 8 files, 1317 insertions
2. `52053f4` (May 25) — Add references.html — CWNA-108 chapter map, 802.11be deep refs, EP1 cross-links — 9 files, 245 insertions
3. `fc4b725` (May 25) — Add assessments.html — 46 phase questions + 30-question CWNA-108 mock — 9 files, 945 insertions

Total: **~2,500 lines of HTML across 11 files**

## DESIGN CONVENTIONS

- **Style**: NYT journal (white, Libre Baskerville serif body + Libre Franklin sans-serif labels)
- **Layout**: Single column max 900px (phases) / 1000px (assessments) / 1100px (references)
- **Card pattern**: `border-top` color tag per phase (matches phase color)
- **Socratic blocks**: Italicized, colored left border, light tinted background
- **Lab boxes**: Tinted background, uppercase title, mono code samples
- **Answer reveals**: `<details><summary>` collapse blocks for assessments only
- **Nav on every page**: Home · Hub · prev phase · Day# · next phase · References · Assessments
- **Footer**: Always `© 2026 KhKLab · <page name>`
- **Privacy footer**: "WLPC corpus: private reference only, not reproduced" where relevant

## EOD HTML PLAN (for tomorrow)

Should be added to `5320-onboarding/main` as `session_summary_20260525.html` per NYT convention.
Link from all 3 landing pages (index.html, index-nyt.html, index-harpers.html).
Tag color: NEW shade (suggest gold `#a16207` or chocolate `#78350f`) — not yet used in EOD ledger.

EOD HTML should cover:
1. Repo creation (gh CLI install via brew, gh auth login, gh repo create)
2. The 6-phase curriculum scope (34 days, ~55–60 hours)
3. References integration (CWNA-108 + 802.11be 15-feature table + EP1 9-link table)
4. Assessments (46 phase Q + 30-Q CWNA mock = 76 total)
5. Live URL + commit hashes
6. Cross-link from the EP1 work in 5320-onboarding (bidirectional)

## SPRINT BACKLOG IMPACT

- New ongoing sprint: **802.11 mastery curriculum execution** — Phase 1 Day 1 starts when ready (no fixed date — picks up any day)
- Cross-link: Future EP1 work in 5320-onboarding should reference back to wifi-80211-mastery references.html for the theory side
- No code yet — all study HTML, lab exercises require AP3000 + 5320 EXOS + MacBook Wireshark (no new hardware)

## RELATED MEMORY FILES

- `reference_wlpc_corpus.md` — T1/T2/T3 corpus tiers
- `reference_wlpc_2026_troubleshooting.md` — 6-process framework + 30-row table
- `reference_revolution_wifi_capacity_planner.md` — airtime methodology used in Phase 5
- `reference_wlanpros_checklists_v5.md` — Top 20 used as Phase 1/6 checkpoint
- `feedback_eod_html_format.md` — NYT style was confirmed May 12; this repo follows that
- `project_5320_onboarding.md` — XIQ-EP1 work this curriculum cross-links to
- `reference_5320_pages_source.md` — Pages source rule (main root)
