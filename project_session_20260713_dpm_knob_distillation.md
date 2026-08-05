---
name: Session 2026-07-13 — Claude File Audit + DigitalTwinEngine 15→12 Knob Distillation
description: Full home-directory Claude file/project audit, security re-verification of cisco-en-cli-agent findings (still open), WiFi 7 twin status check (never started), Golden Parameters clarified as GS Convergence Protocol, 15-param AP bridge distilled to 12 optimizer knobs with sourced suggested values, full Wireshark 4-way handshake study. EOD HTML LIVE: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/docs/session_summary_20260713_dpm_pcap_deepdive.html (commit 92ec4a3).
type: project
originSessionId: cisco-en-cli-agent-20260713
---
# Session — July 13, 2026 (in `/Users/khukhan/Projects/cisco-en-cli-agent`)

## Part 1 — Full Claude-related file/project audit
User asked to scan all directories for Claude-related files and projects. Findings:
- Git repos with `.claude/`+`CLAUDE.md`: `cisco-en-cli-agent`, `eve-ng-lab-platform`, `deep-research-engine`, `xiq-ep1-intelligence-engine`, `voss-fabric-migration`, `cricket-guide-gui`, `cricket-guide`, `5320-onboarding-agent`
- Non-git folders: `~/Claude/Scheduled`, `~/Claude_Projects/tutorial_agent` (+ a `copy` duplicate)
- ~150+ dialogue/session `.docx`/`.pdf` transcripts scattered in `~/Downloads` and OneDrive
- **Discovered the real `MEMORY.md`** lives at `/Users/khukhan/.claude/projects/-Users-khukhan/memory/MEMORY.md` — a *different* project scope (home-dir level) than this repo's own (empty) memory dir. This is the file the global CLAUDE.md's "Active Alerts" briefing instruction refers to.

### ⚠️ Security finding (verified against live code, not just memory)
`security_assessment_cisco_en_cli_agent.md` (dated 2026-04-02) claimed PAT revocation was "pending." Checked actual files in this repo on 2026-07-13 — **none of the critical fixes landed in 3+ months**:
- T-001: GitHub PAT (`ghp_gLforg5V...`) still live in `.env`
- T-002: `auth.py:87` still does plaintext password comparison (`_hash()` defined but never called)
- T-004: `search.py` still builds SQL via f-string interpolation (SQL injection open)
- T-007: `.streamlit/config.toml` still has `enableCORS=false`, `enableXsrfProtection=false`

Also found (unrelated repo): `~/cricket-guide-gui/.git/config` has a live GitHub PAT embedded in the remote URL in plaintext.
**Action needed:** revoke both tokens at github.com/settings/tokens; user has not yet confirmed remediation this session.

## Part 2 — Active Alerts briefing delivered
Per global CLAUDE.md instruction, briefed all concurrent-session threads from `MEMORY.md`: deep-research-engine (Phase 1 blocked), 802.11 Mastery curriculum (Phase 1 Day 1 not started), Sprint C EP1 Traverser (xcloud blocked), multi-LLM framework (plan only), GitHub org ambiguity (unresolved), RADIUS/VLAN sprint (blocked on Karl), WiFi 7 Digital Twin (see below).

## Part 3 — WiFi 7 Digital Twin (ns3-Sionna track) — status check
User picked this thread. Verified: `/Users/khukhan/wifi-7-twin/` **does not exist**. `project_wifi7_twin_sprint.md` said "Phase 0 starts tomorrow (June 17)" — never happened, ~4 weeks later still NOT STARTED. User did not proceed with starting it (dismissed the resume-point question) — pivoted instead to asking about "RF_ParameterConfig for Golden Parameters."

## Part 4 — "Golden Parameters" clarified
Searched for literal string `RF_ParameterConfig` — zero hits anywhere on disk or in memory. User confirmed the real reference: **wifi_dpm 60 design parameters**. Traced it to the correct concept: **"Golden Parameters" = GS (Golden Set) Convergence Protocol**, the Sprint 4 capstone of the **DigitalTwinEngine** project (distinct from the WiFi 7 ns3-Sionna twin — this is the AP3000 live-calibration twin in `/Users/khukhan/5320-onboarding-agent/live_monitor/`), designed June 9 2026 in `project_digital_twin_live_ap_calibration.md`.

### Confirmed 60-param structure
- 60 total DPM parameters (7 sections A–G, see `project_wifi_dpm.md`)
- 45 static/context (site geometry, clients, apps — never from AP)
- 15 dynamically bridged from live AP SSH polling
- 12 "optimizer knobs" (subset/overlap of the 15) tuned by `OptimizerAgent`/`FederatedTwinAgent`

### Build status verified against actual repo (not just memory)
Checked `live_monitor/*.py` files + `git log --oneline -- live_monitor/`:
| Sprint | Status |
|---|---|
| 1 (APCollectorAgent, dashboard, widget) | ✅ built |
| 2a (SQLite storage, session HTML, /health) | ✅ built |
| 2e (PacketCaptureAgent) | ✅ built |
| 2b (`knob_panel.py` gauges) | ❌ not built |
| 2c (`federated_debug.py`) | ❌ not built |
| 2f (pcap rolling loop) | ❌ not built |
| 3 (`SimulatorAgent`/`CalibrationAgent`/`OptimizerAgent`/`FederatedTwinAgent`) | ❌ not built |
| **4 — GS Convergence Protocol ("Golden Parameters")** | ❌ **design-only, zero code** |

Last commit touching this repo: 2026-06-30 (RADIUS/VLAN sprint — unrelated track). Last true digital-twin commits: ~June 9–10.

## Part 5 — 15→12 optimizer knob distillation table (delivered to user)

**Environment:** AP3000 (HiveOS/IQ Engine OS, WiFi 6/802.11ax, dual-band only — wifi0=2.4GHz, wifi1=5GHz, no 6GHz), lab unit **AH-556680**, hardware ceiling 2×2 MIMO / max MCS 11 / max 80MHz(5G) / 40MHz(2.4G). Archetype: lab/small-enterprise.

| # | Knob (`OPT_SEARCH`) | Bridge field | Match | Suggested value | Reference | Note |
|---|---|---|---|---|---|---|
| 1 | `txPwr` | tx_power_dbm | EXACT | 18 dBm/chain (current); ACSP ceiling 20 dBm, floor 5 dBm | AP3000 SSH June 5 (`One Chain EIRP power=22.00dBm(18dBm+4dBi)`) · WLANPros Top20 #13 (target 13 dBm 5GHz) | Current sits above WLANPros' 13 dBm target — candidate to step down |
| 2 | `bw5g` | channel_width_mhz | EXACT | 80 MHz (hardware max on wifi1) | AP3000 SSH June 5 · WLANPros Extended #27 (160MHz "not recommended") | Already at WLANPros-endorsed ceiling |
| 3 | `maxCli` | max_clients | EXACT | 100 (profile ceiling, not a target) | AP3000 SSH June 5 · WLANPros Top20 #20 (<40/radio standard density) | Non-binding at lab client counts |
| 4 | `cwBe` | cw_min_be | EXACT | CWmin=4 / CWmax=6 (as reported) | AP3000 SSH June 5 radio profile | ⚠ Unconfirmed if raw CW or 2ⁿ exponent — verify before use |
| 5 | `txopBe` | wmm_txop_be | EXACT | 0 | AP3000 SSH June 5 (`txoplimit=0`) | Confirmed as-is |
| 6 | `atFair` | lb_airtime_limit_pct | EXACT | 4% (current) | AP3000 SSH June 5 · WLANPros Extended #54 ("not recommended") · calibration_theory (capMult ×1.38 empirically uncertain) | Conflicting guidance — flag for Sprint 3 debate |
| 7 | `mcsFloor` | weak_snr_threshold_db | PROXY | 15 dB | AP3000 SSH June 5/9 radio profile | Shares field with `minRSSI` |
| 8 | `minRSSI` | weak_snr_threshold_db (same as #7) | PROXY | 15 dB | same as #7 | Don't conflate with WLANPros #11/#12 RSSI coverage targets (-65/-67dBm) — different parameter |
| 9 | `dtim` | beacon_period_ms | PARTIAL | Beacon=100ms confirmed; DTIM not exposed — assume DTIM=1 | AP3000 SSH June 5 (beacon) · WLANPros Extended #21/#22 (DTIM default=1) | DTIM value is an assumption, not measured — biggest gap in the bridge |
| 10 | `rxSop` | — | NOT EXPOSED | none — leave at chipset default | No AP3000 CLI field; not named in local WLANPros/Revolution corpus | Don't fabricate a number; needs datasheet or live test |
| 11 | `fragThr` | — | NOT EXPOSED | 2346 bytes (effectively disabled) | IEEE 802.11 `dot11FragmentationThreshold` MIB standard default | Standard default, not AP3000-verified |
| 12 | `rtsThr` | — | NOT EXPOSED | 2347 bytes (effectively disabled) | IEEE 802.11 `dot11RTSThreshold` MIB standard default | Same caveat |

**Summary:** 6/12 knobs hardware-confirmed, 2 share one proxy field, 1 is a stated assumption, 3 have zero AP3000 visibility and need either a vendor datasheet or the "Degraded RF Injection" test plan (already logged in `project_digital_twin_live_ap_calibration.md`) before they belong in the optimizer's search space.

## Pending / Next
- User has not yet acted on the PAT revocation (T-001, both cisco-en-cli-agent and cricket-guide-gui repos)
- Sprint 2b (`knob_panel.py`) is the actual next unbuilt step before any Golden Set/Sprint 4 work can start
- EOD HTML publication for this session — pending, this memory file is the source content

## Part 6 — Clarifying the "3 other parameters" + full 60-param list (appended)

User asked "where are the 3 other parameters" after the 12-knob table. Corrected an earlier conflation: there are **three distinct "15" lists** in the DigitalTwinEngine memory, not one:
1. **June 5 original "15-Parameter Bridge"** — AP telemetry → simulator input mapping (station count, channel width, CRC%, noise floor, SNR, Tx/Rx CU%, interference CU%, MCS proxy, per-client chan-width, ACSP neighbor count, neighbor RSSI, collision_state, peak Tx Mbps, VLAN distribution)
2. **"15 Search Knobs → AP CLI"** (later June 5 dialogue) — DPM-param-to-AP-CLI table, 15 numbered rows **+ 3 appended rows** ("+3": Guard Interval, MCS Floor, DTIM interval) = 18 total entries
3. **"12 Optimizer Knob Coverage"** (June 9) — the `OPT_SEARCH` JS variables (txPwr, bw5g, maxCli, cwBe, txopBe, mcsFloor, dtim, atFair, minRSSI, rxSop, fragThr, rtsThr) checked against AP3000 field coverage — this is the list the 15→12 table used

**Answer:** Of list 2's "+3" (Guard Interval, MCS Floor, DTIM interval), MCS Floor and DTIM interval are already covered in the 12-knob table (rows `mcsFloor`, `dtim`). **Guard Interval is the one missing parameter** — it is NOT one of the 12 `OPT_SEARCH` knobs at all; it's a separate AP3000-confirmed hardware fact (`Short guard interval=enabled`, 400ns, confirmed June 5 SSH) that was never promoted into the optimizer's tunable set.

### Full 64-parameter DPM list (memory shorthand rounds to "~60"), from `project_wifi_dpm.md`
- **A — Site & Environment (10):** Site type, Coverage area (m²), Number of floors, Ceiling height, Wall type, RF environment type, Noise floor (dBm), Available airtime %, Regulatory domain, U-NII bands available
- **B — Client Devices, per type (12):** Client device type, 802.11 protocol, Tx antenna chains, Rx antenna chains, Antenna gain 2.4GHz, Antenna gain 5GHz, Max channel width, Frequency band support, Device quantity, Concurrent association %, Concurrent active %, Concurrent limit mode
- **C — Applications, per app (6):** Application type, Throughput SLA, Transport, Latency sensitivity, Average frame size, Background app flag
- **D — Network Design (13):** AP model, 5GHz channel width, 6GHz channel width, Client band distribution %, Association limit per radio, Number of enabled SSIDs, MBR 2.4GHz, MBR 5GHz, MBR 6GHz, RF coverage tier, Capacity for growth %, 2.4GHz radio limit, Growth band preference
- **E — Channel Planning (7):** 2.4GHz channels (1/6/11), 5GHz channel set, DFS usage, 6GHz PSC channels, CCI threshold, ACI threshold, Consistent channel bonding
- **F — Security & SSID (7):** Security type per SSID, 802.11r FT, 802.11k Neighbor Report, 802.11v BSS Transition, 802.11w MFP, VLAN per SSID, Client isolation
- **G — RRM & AutoRF (9):** RRM mode, XIQ cloud RRM, Broadcom per-AP AutoRF, AutoRF CCI trigger, AutoRF retry rate trigger %, AutoRF channel util trigger %, TX power min, TX power max, Dynamic channel width

Total: 10+12+6+13+7+7+9 = **64** (vs. the "~60" shorthand used elsewhere in memory/index entries).

## Part 7 — EP1 Radio Settings cross-check + EN doc discovery + IEEE standard reconciliation (appended)

User pasted a raw EP1 "Radio Settings" config panel excerpt. Cross-checked against today's accounting:
- **Beacon period (100 TUs)**, **short guard interval**, **OFDMA**, **BSS Coloring** — already tracked (dtim knob, GI gap, 15-Bridge rows)
- **A-MPDU, Frame Burst, Transmit Beamforming, TWT** — confirmed via AP3000 SSH but only logged as "supplemental params," never promoted into the 64-DPM/12-knob/15-bridge tables
- **Preamble** — not tracked anywhere at all

### Source document identified
The pasted text is verbatim from **`Extreme_Platform_ONE_Networking_25_9_0_User_Guide.pdf`, pages 358–359**, section "Configure AP Templates → Radio Settings" (steps 9–17 of "Configure a Radio Profile"). File cataloged in `project_ep1_transition_resources.md`, also present at `5320-onboarding-agent/docs/rag-corpus/`.

Extracted definitions (verbatim from EN doc):
- **BSS Coloring**: "numerical identifier of the BSS... 802.11ax radios are able to differentiate between BSSs using BSS color identifiers when other radios transmit on the same channel." Enable + numeric value entry, no explicit range stated in doc (802.11ax standard: 6-bit field, 0–63).
- **Beamforming**: "Select Enable Transmit Beamforming to improve data transfer rates for directional signal transmission processing." One-line description, Enable/Disable only.
- **Preamble**: short preamble negotiated with client if supported, else falls back to long. Short saves time/throughput; long gives receiver more sync time in noisy environments.
- **Beacon Period**: default 100 TUs, **configurable range 40–3500 TUs** (new number, not previously on record).
- **Guard Interval**: **CORRECTION — default is 800ns**, not 400ns. Earlier accounting only had the AP3000's *current configured state* (short GI enabled = 400ns), not the underlying protocol default. 400ns is the optional "short" setting for smaller/less-reflective spaces; 800ns is standard for large/echoing environments (warehouses, stadiums, outdoors).
- **AMPDU**: combines data frames into fewer larger frames, reduces overhead, "generally increases performance."
- **Frame Bursts**: bursts up to 3 packets using inter-frame wait intervals without releasing the transmission medium.

### Major finding — most of this was ALREADY properly documented
Checked `5320-onboarding-agent/docs/dt_parameter_reference.html` (built June 9, §7/§8 "Feature Flags: IEEE 802.11 Standard") — it **already has correct IEEE clause citations** for BSS Color, Beamforming, A-MPDU, TWT, Short GI, DFS, A-MSDU, MU-MIMO, OFDMA DL/UL. Only **Preamble** and **Frame Burst** are genuinely missing from that reference and from all memory — confirmed via grep, zero hits for either.

### IEEE vs. RFC clarification (important — user asked for "RFCs")
BSS Coloring, Beamforming, Preamble, GI, A-MPDU, TWT, Frame Burst are **IEEE 802.11 standard/vendor features, NOT IETF RFCs**. RFCs are a different standards body covering internet protocols (unlike VOSS/SPB which genuinely does cite an IETF RFC — RFC 6329 — because that's a routing protocol, not a PHY/MAC feature). No RFC exists for these Wi-Fi radio parameters.

### Verified IEEE clause references (cross-checked directly against local standard PDFs, not just memory)
| Param | Clause | Verified how |
|---|---|---|
| BSS Color | 802.11ax-2021 §26.17; 802.11-2020 §26.17.3 | Already in dt_parameter_reference.html |
| Beamforming | 802.11ac §22.3.11.8; 802.11ax §27.3.11 | Already in dt_parameter_reference.html |
| **Preamble** | **802.11-2020 §16.2.2.3** (Short Preamble subfield, Clause 16 ERP/HR-DSSS PHY) | **NEW — verified today by converting and grepping the full 802.11-2020.pdf standard** (`Downloads/WLPC Troubleshooting/Support Documents/802.11-2020.pdf`) |
| **Frame Burst** | **No IEEE clause — proprietary Aerohive/HiveOS term.** Closest formal concept: TXOP (Transmission Opportunity) under EDCA, originally 802.11e, folded into 802.11-2020 Clause 10 | **NEW — confirmed zero hits for "frame burst"/"frameburst" in the full 802.11-2020 standard text; it will never appear in an IEEE standard search** |
| A-MPDU | 802.11n-2009 §9.7.2; 802.11-2020 §9.7.2 | Already in dt_parameter_reference.html |
| TWT | 802.11ax-2021 §26.8; 802.11-2020 §9.4.2.200 | Already in dt_parameter_reference.html |
| Guard Interval | 802.11n-2009 §20.4.7.10; 802.11-2020 §19.3.11.10.2 | Already in dt_parameter_reference.html — default corrected to 800ns (see above) |

### RAG/corpus confirmation (user asked to check if RFCs/IEEE docs already exist locally)
Confirmed present at `/Users/khukhan/Downloads/WLPC Troubleshooting/Support Documents/` (tier T1, per `reference_wlpc_corpus.md`) and mirrored into `5320-onboarding-agent/docs/rag-corpus/`:
- `802.11-2020.pdf` — full consolidated IEEE base standard (converted + grepped in full today to verify Preamble clause)
- `802.11be - Wi-Fi 7.pdf` — academic survey paper on EHT covering BSS Color/Beamforming/Preamble evolution into Wi-Fi 7 (Coordinated Beamforming, Coordinated Spatial Reuse)

No new documents needed — the correct standards were already in the corpus; today's work closed the gap on cross-referencing Preamble and Frame Burst specifically.

### For EOD HTML — must include
All of Part 6 + Part 7: the corrected 12/15/18-knob accounting, the EN doc citation (page 358-359), the corrected GI default (800ns), the 2 genuinely-new gaps (Preamble, Frame Burst) vs. the 6 already covered by `dt_parameter_reference.html`, and the IEEE-vs-RFC clarification.

## Part 8 — Global CLAUDE.md best-practices audit + patch (appended)

User provided `/Users/khukhan/Downloads/Khursheed's Cheat Sheet_Claude Code Items for Consideration in Globals 1.docx` — confirmed this is the raw 11-post LinkedIn/social source material behind `/Users/khukhan/.claude/CLAUDE.md` (the "16-source glossary" from the April 6 global-config synthesis).

**Verification result:** ~95% of the cheat sheet was already faithfully synthesized into the global CLAUDE.md (memory persistence, token optimization tricks, Karpathy KB pattern, power commands, `.claude/` anatomy, quality gates all present and matched almost verbatim).

**4 gaps found and patched into `/Users/khukhan/.claude/CLAUDE.md` on 2026-07-13:**
1. `.claude/plugins/` row added to the Project Anatomy table — confirmed as an actually-in-use feature on this machine (`~/.claude/plugins/marketplaces/claude-plugins-official` exists), previously undocumented in the anatomy table
2. Added explicit "two scopes" sentence: project `.claude/` (committed, team-shared) vs. `~/.claude/` (personal, global)
3. Added a Hook Events reference table (SessionStart, UserPromptSubmit, PreToolUse/PostToolUse, SessionEnd) — previously only mentioned contextually
4. Added the message-count heuristic ("~30 messages → 50,000+ tokens, start fresh every 15-20") alongside the existing %-based context thresholds

**Additional best practices given (not in cheat sheet or CLAUDE.md, from direct operating knowledge):**
- Don't manually `/compact` preemptively — this harness auto-summarizes near the context limit; premature manual compaction can discard detail
- Parallelize independent tool calls in one turn rather than serially
- Verify claims against live state before acting — memory/docs can go stale (this session's cisco-en-cli-agent security re-check is the live example: April assessment said PAT revocation was "pending," but the token was still exposed in `.env` 3+ months later)
- Ask before risky/irreversible actions even under autonomous framing

### For EOD HTML — additional item
Include the CLAUDE.md audit: what was verified, the 4 patched gaps, and the additional best-practices list above.

## Part 9a — claude-global-config repo synced + pushed (appended)
Repo `khursheedkhanaiforgood-ai/claude-global-config` existed on GitHub but was never cloned locally on this machine. Cloned to `/Users/khukhan/claude-global-config`. Found its `CLAUDE.md` was stale since the 2026-04-06 synthesis (missing 3+ months of local drift, not just today's 4 additions). Synced full current `~/.claude/CLAUDE.md` → repo, committed (`b4aa805`), pushed to `main`. **Not yet done:** repo's `synthesis/` subfolder (5 topic files) is now stale relative to the new CLAUDE.md — separate content-authoring task, not a simple sync, left pending.

## Part 9 — Capability check: video content (appended)
User asked whether I can review YouTube videos or LinkedIn posts referencing videos. Answer given: no video/audio transcription or viewing tool is available in this session's toolset. `WebFetch` (if loaded) can retrieve a webpage's text/HTML content, but cannot watch or transcribe video/audio — and LinkedIn/YouTube pages typically don't expose transcript text via a plain fetch (LinkedIn restricts unauthenticated scraping; YouTube's video content isn't in the page HTML). If the user pastes a transcript or the post's text content directly, that can be reviewed normally.

## Part 10 — README + synthesis/ studied, high-leverage items integrated back into CLAUDE.md (appended)

User asked me to study the `claude-global-config` repo's `README.md` and all 5 `synthesis/*.md` files, and whether the updated CLAUDE.md should integrate them.

**Finding:** the synthesis docs contain substantially more actionable detail than CLAUDE.md reflected — several high-leverage items were dropped entirely during condensing, not summarized:
- `05-quality-gates.md`: Etzioni's Ownership Test (AI coach vs. ghostwriter), Taggart's 3 sharpest concrete rules (strongly-typed languages as hallucination guardrail, security as separate explicit phase, review every line), and a **Security Non-Negotiables table** (SQLi/hardcoded secrets/weak passwords/CORS-XSRF/command+path+template injection) that — notably — would have flagged every single finding in today's cisco-en-cli-agent security re-check (live PAT, plaintext passwords, f-string SQL, disabled CORS/XSRF) had it been loaded
- `02-claude-code-power-commands.md`: "The Power Trio" (Worktrees + `/batch` + Hooks as one combined system), `/remote-control`, `/multi-plan`+`/multi-execute`
- `03-architecture-patterns.md` / `04-knowledge-base-pattern.md`: career-ops quality-threshold-gate pattern, verification loop commands, Karpathy→cisco-en-cli-agent specific mapping table — left intentionally in synthesis/ as deep reference rather than pulled into CLAUDE.md (README's own architecture: CLAUDE.md = condensed summary, synthesis/ = deep reference)

**Recommendation given and approved:** integrate selectively, not merge everything — CLAUDE.md's own ≤200-line discipline matters. Added: a pointer sentence at the top of CLAUDE.md directing to `synthesis/` for full depth; the Ownership Test + Taggart's 3 rules + Security Non-Negotiables table in Quality Gates; the Power Trio one-liner + `/remote-control` + `/multi-plan`/`/multi-execute` in Key Power Commands.

**Commits:** `b4aa805` (initial sync — repo was stale since 2026-04-06, missing 3+ months of local drift) then `e461ce8` (this integration pass). Both pushed to `khursheedkhanaiforgood-ai/claude-global-config` main.

### For EOD HTML — additional item
Include: the README/synthesis study, the finding that Security Non-Negotiables table pre-existed in the repo and would have caught today's cisco-en-cli-agent findings, and both commit links (b4aa805, e461ce8).

## Part 11 — WMM/EDCA (Contention Window, AIFS, TXOP) deep dive (appended)

User pasted the EP1 "WMM QoS Settings" panel (4 Access Categories × 5 fields = 20 total: CWmin, CWmax, AIFS, TXOP Limit, No ACK for Voice/Video/Best-effort/Background).

### Cross-check against the 12-knob list
Only **2 of 20** are in the 12-optimizer-knob list: `cwBe` (Best-effort CWmin) and `txopBe` (Best-effort TXOP), both scoped to Best-effort only. Voice, Video, and Background are entirely untracked; CWmax/AIFS/No-ACK for Best-effort are also untracked.

### Key finding — resolved an old ambiguity
EP1 panel field headers show "(1-15)" range for CWmin/CWmax — confirms these are **exponent values** (CW = 2ⁿ−1), not raw CW. Converting the pasted defaults and checking against the industry-standard WMM/802.11e default EDCA table: **all 4 ACs match exactly** (Voice 3/7, Video 7/15, Best-effort 15/63, Background 15/1023; TXOP 1504/3008/0/0 also match). This resolves the earlier ⚠ flag on the 12-knob table's `cwBe` row (AP3000's SSH-confirmed CWmin=4/CWmax=6 are now confirmed as exponents) and confirms the AP is running **stock WMM defaults**, not custom tuning.

**Background (BK) access category had never been documented anywhere in memory before this EP1 panel** — first-time confirmation.

### Definitions given
- **Contention Window (CW)**: random backoff range [0,CW]; resets to CWmin on success, doubles toward CWmax on collision (binary exponential backoff). Smaller CW = higher priority.
- **AIFS**: minimum idle time before backoff even starts (AIFS = AIFSN × slot_time + SIFS), per-AC. Lower AIFSN = starts contending sooner.
- **TXOP Limit**: max duration held after winning contention, for bursting multiple frames. Voice/Video get non-zero (burst allowed); Best-effort/Background = 0 (no bursting).
- Standard reference: 802.11-2020 (originally 802.11e-2005), EDCA.

### EN's actual recommendation (found in the same EP1 User Guide, p.363, "Configure WMM QoS Settings")
"If necessary, modify the default settings..." — **EN's own guidance is to leave WMM CW/AIFS/TXOP at defaults unless there's a specific reason to change them.** One explicit exception: EN recommends **enabling No-ACK for Video** ("lost packets in streaming video go unnoticed, and retransmissions are unnecessary"). WLANPros checklist has no per-parameter guidance here — only a generic "WMM: ON" item (#43).

### cwBe/txopBe tunability — confirmed yes, with guardrails
These 2 are legitimately meant to be search knobs (Best-effort = lowest-stakes AC to experiment on, unlike latency-critical Voice/Video). Guardrails given: don't shrink `cwBe` below Video's window (7/15) — would let BE traffic out-compete Video, defeating WMM prioritization; keep any non-zero `txopBe` well below Voice/Video's burst allowances to avoid BE hogging airtime ahead of latency-critical traffic.

### Critical honesty check — does the 12-knob list actually correlate with QoE?
User asked me to confirm the 12 tunable parameters are "the ones which affect QoE." **Answer: not confirmed, and importantly this has never been empirically tested.** Breakdown:
- **~6 have clear, direct, mechanistically-established QoE links**: txPwr, bw5g, atFair (strong); maxCli, cwBe, txopBe (conditional/scoped — cwBe/txopBe only affect Best-effort-classified traffic, not the whole network)
- **2 collapse into 1**: mcsFloor and minRSSI map to the same underlying AP field (weak_snr_threshold_db) — not independently tunable, so it's really 11 knobs not 12
- **1 is a proxy, not the real variable**: dtim tracks beacon_period_ms, not the actual DTIM interval (not independently observable on AP3000)
- **3 are non-functional on this specific AP3000**: rxSop (not exposed at all via CLI), fragThr/rtsThr (exposed only as inert standard defaults — effectively disabled, no practical effect on typical traffic)

**Root cause:** the 12-knob list is a design hypothesis from the June 9 architecture dialogue, not a validated sensitivity ranking. The mechanism that would actually confirm which knobs move QoE — the Monte Carlo sensitivity oracle (∂QoE/∂param_i) — is part of Sprint 3's `SimulatorAgent`, which (per Part 4's build-status check) has never been built or run. No sensitivity analysis has ever executed against these knobs.

### For EOD HTML — additional item
Include: the full WMM/EDCA definitions, the exponent-resolution finding (resolves prior ambiguity), EN's actual recommendation (defaults + No-ACK-for-Video exception), the cwBe/txopBe tunability guardrails, and — most importantly — the QoE-linkage honesty check (12-knob list is unvalidated hypothesis, not confirmed fact; effectively 11 independent knobs with 3 currently inert on this hardware).

## Part 12 — Full 3-band radio profile comparison (6GHz, 5GHz, 2.4GHz) + EN/internet verification (appended)

User pasted the full `radio_ng_11ax-6g`, `radio_ng_11ax-5g`, and `radio_ng_11ax-2g` EP1 radio profile config panels (AP3000 Template). Built a section-by-section comparison matrix across all 3 bands and cross-referenced against the 12/15/64/WMM accounting.

### 3 items checked against EN reference doc + public web search
1. **ZeroWait DFS** — confirmed definition (EN doc p.355): "dedicate a single antenna chain to quickly identify a usable DFS channel... a 4x4 AP become a 3x3 AP... only available for 3- or 4-stream APs." **AP3000 is 2×2 — ZeroWait DFS is not actually usable on this hardware**, same "template shows inert features" pattern as 6GHz SP/LPI.
2. **SP/LPI (6GHz Power Modes)** — confirmed (EN doc p.116, 5492-5519): LPI=Low Power Indoor, SP=Standard Power requiring AFC geo-location authorization, explicitly tied to AP5020/AP5050/AP4020 only — confirms this field is inert on AP3000 (no 6GHz radio hardware at all).
3. **Missing Short-GI/Beamforming toggles on 6GHz profile** — plausible explanation via web search (not EN-confirmed): 802.11ax replaces the legacy binary 400/800ns GI toggle with a 3-value HE scheme (800/1600/3200ns); 6GHz is HE-only (no legacy fallback) while 5GHz/2.4GHz retain legacy modes where the classic toggle applies. Flagged as well-supported inference, not verified fact. Sources: [Cisco 802.11ax GI doc](https://www.cisco.com/c/en/us/td/docs/wireless/outdoor_industrial/iw9167/software/configuration/261x/b-iw9167-ap-scg-261x/g-ap-mode-config-17-11/c-80211ax-1600ns-and-3200ns-guard-interval.html), [Silicon Labs 802.11ax GI/LTF](https://docs.silabs.com/wifi91xrcp/latest/wifi91xrcp-developers-guide-wifi-configuration/802-11-ax-gi-and-ltf).

### 3-band comparison matrix (summary — full table given to user in-session)
- **Identical across all 3 bands:** TX Power (20/5/18 dBm), Max Clients (100, range 1-255), Weak Signal SNR threshold (15dB), Safety Net (15s), Preamble/Beacon (100 TUs), AMPDU/FrameBurst/OFDMA/BSSColor/TWT, Backhaul Failover (2s/30s), WMM QoS 4-AC table, Sensor Scan Dwell (1200ms)
- **2.4GHz-only:** Region/Channel-Model selector (USA=3ch[1,6,11]/Europe=4ch[1,5,9]) — confirms WLANPros Top20 #2 is literally built into the EP1 GUI as the USA default; Channel width = 20MHz only (matches WLANPros #3/#4)
- **5GHz-only:** Full DFS section + ZeroWait DFS, Radio Load Balancing (dual-5G, N/A on AP3000's single 5GHz radio), legacy client deny toggle
- **6GHz-only:** SP/LPI power mode (inert on AP3000), PSC channel display, 160MHz channel width option; **absent:** DFS section, High Density Config, Band Steering, Client Load Balancing, Short GI toggle, Beamforming toggle
- **2.4GHz + 5GHz share (6GHz lacks):** High Density Configuration, Band Steering, Client Load Balancing (CRC 30%/RF-Interference 40%/Airtime 4%), Short GI toggle, Beamforming toggle

### For EOD HTML — additional item
Include: the full 3-band matrix, the 3 EN/web-verified findings (ZeroWait DFS inert on 2×2 AP3000, SP/LPI inert on AP3000, HE-GI-scheme explanation for 6GHz toggle absence), and the 2.4GHz Region/Channel-Model finding confirming WLANPros #2 is a GUI default, not just a recommendation.

## Part 13 — Cisco RRM White Paper (logic, not values) + 802.11be column + EN vs. Cisco RRM deficiency analysis (appended)

User asked: is there a Cisco/Juniper/industry document showing the **logic** behind radio parameter configuration (not just recommended values)? Also asked to add an 802.11be column to the Part 12 3-band matrix, keeping notes.

### Found: Cisco RRM White Paper already downloaded locally
`/Users/khukhan/Downloads/WLPC Troubleshooting/Xtra Additional Documents/Cisco_RRM_White_Paper.pdf` — converted and read the DCA and TPC algorithm sections directly (not just referenced by name in `calibration_rag_sources.html`, which was already known to catalog Cisco/Juniper design guides by URL).

**DCA (Dynamic Channel Assignment) logic:**
- Single **Cost Metric (CM)** combining CCI + foreign-channel interference + non-Wi-Fi noise + QBSS channel load
- **CPCI** = worst-performing AP selected first, alternating with random selection each iteration until the whole group is processed
- Evaluation scope bounded to 1st-hop + 2nd-hop neighbors only (2nd-hop channels evaluated but never changed — bounds blast radius)
- **NCCF** gating: a channel change is only recommended if the CPCI improves AND its neighbor group doesn't regress
- **DCA Sensitivity**: tunable 10-20dB hysteresis threshold (default "medium" ≈10dB) to prevent flip-flopping on bursty RF

**TPC (Transmit Power Control) logic:**
- `Tx_ideal = Tx_max + (Power_Threshold − RSSI_3rd_neighbor)` — power derived from the 3rd-loudest **AP-to-AP** neighbor (not client RSSI), tied to 3 = number of non-overlapping 2.4GHz channels
- Default threshold -70dBm (was -65dBm before 4.1 MR1), valid range -50 to -80dBm, good-practice range -68 to -75dBm

### EN vs. Cisco RRM — deficiency analysis given
Real, specific gaps surfaced by comparison: (1) EN uses 3 independent flat thresholds (Interference/CRC/Util all @35%) vs. Cisco's single weighted Cost Metric; (2) no documented neighbor-impact gating (no NCCF equivalent) in EN's ACSP/DCS; (3) no documented worst-first/CPCI-style prioritization; (4) **no documented dynamic TX-power formula for EN** — only static bounds (Max/Floor/MaxDrop) vs. Cisco's explicit deterministic formula; (5) EN's hysteresis is qualitative ("medium") vs. Cisco's quantified 10-20dB.

**Important caveat given, not overclaimed:** this may partly reflect documentation opacity (Cisco publishes an algorithm white paper; EN's public docs describe configurable fields, not XIQ Cloud RRM's actual cloud-side logic) rather than proven algorithmic inferiority. Also noted: Extreme/HiveOS is architecturally distributed (per-AP local scan + cloud check-in) vs. Cisco's centralized WLC/RF-Group-Leader model — a design-philosophy difference with real tradeoffs, not just a maturity gap.

**Relevance:** directly informs Sprint 3's unbuilt `OptimizerAgent` — Cisco's cost-metric + neighbor-gating model is a legitimate alternative architecture to the simpler independent-threshold model implied by everything confirmed about the AP3000 to date.

### 802.11be (WiFi 7) column added to Part 12's 3-band matrix
Sourced from Cisco Meraki Wi-Fi 7 Technical Guide (`/Users/khukhan/Downloads/Wi-Fi 7 (802.11be) Technical Guide - Cisco Meraki Documentation.pdf`, read directly) + the 802.11be survey paper reviewed earlier (Part 7 context). New 802.11be-specific mechanisms found (none configurable on AP3000 — WiFi 6 hardware only, shown for architectural context):
- **320MHz channel width** (6GHz only; 3× in LPI mode, only 1× in US Standard Power mode)
- **4096-QAM (4K-QAM)**: 12 bits/subcarrier vs 10 bits for 1024-QAM — matches the MCS 12/13 / SNR≥36dB fact already in `dt_parameter_reference.html`
- **Preamble Puncturing**: carves out only the interfered 20MHz sub-channel instead of abandoning the whole wide channel; only works above 80MHz, not at 40MHz — extends the Preamble finding from Part 12
- **MLO (Multi-Link Operation)**: entirely new cross-band dimension, not a per-band column value — modes MLSR/EMLSR (single radio) vs. MLMR-STR/nSTR (multi-radio simultaneous/non-simultaneous)
- **New AKM 24/25** for WPA3-Personal + mandatory beacon protection, MLO requires security established across all active links simultaneously
- All other rows (TX power, max clients, WMM, beacon, AMPDU, etc.) carry forward unchanged — no 802.11be-specific delta found

### For EOD HTML — additional item
Include: the Cisco RRM algorithm details (DCA cost-metric/CPCI/NCCF/sensitivity, TPC formula), the EN-vs-Cisco RRM deficiency analysis with the documentation-opacity caveat, and the 802.11be column (320MHz/4K-QAM/Preamble Puncturing/MLO/new AKMs) added to the 3-band matrix.

## Part 14 — QoE qualification methodology (Cisco vs EN) + EN validation-gap deep dive + Aerohive legacy find (appended)

### QoE qualification methodology (desk-research + build-it-and-test)
User asked how to qualify/compare QoE delivered by Cisco vs EN deployments. No empirical comparison exists — everything to date compared documented algorithm logic, not measured outcomes. Two legitimate paths given:
1. **Desk research**: score real deployments (Cisco's own published stadium/HD case studies already in corpus, your own AP3000 lab) against the vendor-neutral WLANPros Top-20 checklist as a common rubric; use Juniper Mist's SLE framework (already cataloged in `calibration_rag_sources.html`) as a third external benchmark.
2. **Build-it-and-test (the real answer)**: extend Sprint 3's unbuilt `OptimizerAgent` with a **pluggable RRM-strategy interface** — Strategy A = Extreme's confirmed independent-threshold logic, Strategy B = Cisco's published Cost-Metric/CPCI/NCCF logic — run both against the identical simulated environment and compare resulting 5D QoE vectors. This is a new architectural addition to the Sprint 3 design that didn't exist before today's Cisco RRM research. **Held for later per user request — not yet added to the formal Sprint 3 architecture.**

### EN validation gap — confirmed, not just suspected
Grepped the entire 33,261-line `Extreme_Platform_ONE_Networking_25_9_0_User_Guide.pdf` for "capacity planning," "predict," "site survey," "design tool," "pre-deployment," "mall," "retail," "dense urban," "shopping" — **zero hits on every term.** Confirmed: EN's own current product documentation contains no predictive capacity-planning or pre-deployment QoE-validation methodology at all.

**What "Client SLA" actually is** (p.362 radio-profile phymode Rate/Success/Usage targets; p.331 user-profile throughput target + Log/Boost-Airtime actions): a **reactive, post-hoc, per-client health-scoring and remediation mechanism** — not predictive. It tells you whether an already-connected client is meeting expectations; it does not tell you, before deployment, whether a planned config will deliver a QoE target for a new site.

**User's own gap named explicitly:** "All I am doing now is setting the values or toggling in the Radio_Profiles... there is NO way for me to validate." Confirmed as an accurate read of the situation, not a misunderstanding.

**Practical bridge given** (usable today without waiting on Sprint 3): (1) manual before/after delta using `live_monitor` (Sprint 1/2a, already built) against a chosen target; (2) WLANPros Top-20 as the pass/fail bar instead of EN's own Client SLA scoring; (3) Revolution Wi-Fi Capacity Planner as the "before" prediction, compared manually against `live_monitor`'s "after" measurement.

**Archetype gap found:** the 5 existing archetypes (Stadium/Enterprise/Hospital/Warehouse/Residential in `project_digital_twin_calibration_theory.md`) do not include "dense-urban retail/mall" — the specific scenario the user asked about. Worth adding as a 6th archetype (distinct traffic profile: bursty POS + BYOD browsing + high transient/unassociated foot-traffic churn).

### Major find — legacy Aerohive High-Density Wi-Fi Design & Configuration Guide (2013)
Web search + WebFetch (PDF saved locally by WebFetch, then converted directly via `pdftotext` since the fetch model couldn't parse the image-heavy PDF) surfaced: **"Aerohive Design & Configuration Guide — High-Density Wi-Fi" by Andrew von Nagy (2013)** — same author as the Revolution Wi-Fi Capacity Planner already in corpus. Directly relevant since AP3000 is Aerohive/HiveOS-heritage hardware.

**Real capacity-forecasting methodology found** (§"Forecasting AP Capacity," p.19-20): `airtime_per_device = throughput_target ÷ max_achievable_throughput`; `devices_per_radio = 80% ÷ airtime_per_device`; `radios_required = (device_count × airtime_per_device) ÷ 80%` — near-identical to Revolution Wi-Fi's methodology, same author, earlier document. Includes a fill-in worksheet (Device Model × Band × Quantity × Airtime → # AP Radios). Explicitly states: **"Use the forecasted AP capacity as input into a predictive RF site survey... The site survey process verifies your AP capacity forecast"** — a genuine forecast→verify loop, done via RF site survey tooling rather than live telemetry.

**SLA remediation mechanism more fully revealed** (§"Client SLA Compliance," p.92-93): current EP1 docs only document 2 remediation actions (Log, Boost Airtime). The legacy Aerohive guide reveals **3**: airtime boost + band steering (2.4↔5GHz) + client load balancing between APs — a broader toolkit than currently documented. Named underlying engine: **"Dynamic Airtime Scheduling"** (per a referenced deeper whitepaper, "SLA Compliance: Wireless Fidelity Achieved," located at webtorials.com archive but not fetchable — 403 error — so its internal algorithm remains unconfirmed). Even the 2013 guide's own conclusion matches current EP1: persistent SLA violations mean "there might be...insufficient capacity" — i.e., the documented fallback is manual capacity redesign, not automation, in both the legacy and current docs.

**Confirming signal that this methodology wasn't carried forward:** `docs.aerohive.com/.../configuring-client-sla.htm` now 301-redirects to a generic `supportdocs.extremenetworks.com` landing page — the specific legacy documentation no longer resolves at its old location.

**Conclusion given to user:** Extreme (via Aerohive) *used to* have a real capacity-forecasting + SLA-monitoring methodology; it does not appear to have been carried forward into current EP1/XIQ documentation at the same depth. No current-day (2026) Extreme-published equivalent to Cisco's stadium/HD design guides was found via web search.

### For EOD HTML — additional item
Include: the QoE qualification methodology (2 paths, pluggable-RRM-strategy idea held for later), the confirmed EN validation gap (zero hits across the entire User Guide for capacity-planning terms), the practical bridge (live_monitor + WLANPros + Revolution WiFi Planner), the new mall/retail archetype gap, and the Aerohive legacy guide find (capacity-forecasting formula, 3-pronged SLA remediation, Dynamic Airtime Scheduling, and the finding that this methodology appears dropped from current EN docs).

## Part 15 — WiFi packet capture primer (appended)

User asked to learn "everything about packet captures on wifi interfaces." Grounded the primer in local corpus resources found for the first time this session: `WLPC Troubleshooting/Support Documents/Wireshark_Filters.pdf` (SANS Institute 802.11 display-filter pocket reference — frame type/subtype table, 4-address header reference, frame control sub-fields) and `WLPC Troubleshooting/Support Documents/WLAN Pi Documents/Linux Commands for Wireshark for WLAN Engineers.pdf` (monitor-mode setup: `airmon-ng`, `iw reg set US` for 6GHz unlock, NIC firmware fix for Data-frame capture, Wireshark root/dumpcap permissions). Also 2 sample pcaps already in corpus: `Capture 08 - RADIUS over Wireless.pcapng`, `Capture 09 - Radius over Ethernet.pcapng` (not yet opened/analyzed).

### Primer content delivered
1. **Monitor vs managed mode** — why 802.11 capture fundamentally differs from wired promiscuous mode; monitor mode locks to one channel at a time (ties to why `CaptureAgent`'s interface selector and the still-pending rolling-loop design must choose a channel per window)
2. **3-tier frame hierarchy** (Management/Control/Data) with type/subtype table from the SANS reference
3. **4-address 802.11 header scheme** — ToDS/FromDS-dependent address meaning, table for all 4 combinations (IBSS/AP→client/client→AP/WDS); 5G-bridge analogy given (DS hop ≈ which interface you're capturing at, like S1/N2 framing differing from radio-bearer framing)
4. **3 capture methods in the user's actual environment**: (a) AP3000 on-device `capture interface`/`filter l2`/`filter l3`/`exec capture remote-sniffer` commands (already documented in `reference_ap3000_command_themes.md`); (b) external monitor-mode capture via WLANPi/laptop (grounded in the WLANPi Linux doc — airmon-ng, 6GHz region unlock, firmware/permission gotchas); (c) the user's own `CaptureAgent` (Sprint 2e, confirmed built) as the productized wrapper around (a)
5. **Encryption/decryption**: WPA2-PSK decryptable in Wireshark only with passphrase+SSID AND a captured full 4-way handshake for that client; WPA3-SAE has no static PSK-to-PMK shortcut (per-session unique PMK) — tied back to why the May 6-8 DHCP/GTK-rotation incident investigations needed fresh handshake captures and `wsec`/`wsec_dump` debug keys
6. **Filter cheat-sheet** from the SANS reference (mgmt/beacon/deauth/data/retry/DHCP filters), noting which ones were already used live during the June 10 packet-capture sprint test plan
7. **Built vs. missing recap**: `CaptureAgent` (Sprint 2e) built; Sprint 2f rolling-loop + `CorrelationAgent` (auto pcap_inspector fault-window flagging) still not built — matches Part 4's Sprint status table

### For EOD HTML — additional item
Include: the WiFi packet capture primer (monitor mode, frame hierarchy, 4-address header, 3 capture methods available in this environment, encryption/decryption constraints, filter cheat-sheet), and the discovery of 2 new local corpus resources (SANS Wireshark filter reference, WLANPi Linux setup guide) plus 2 unanalyzed sample pcaps (RADIUS over Wireless/Ethernet) worth opening in a future session.

## Part 16 — Local pcap inventory + new `/Users/khukhan/pcaps/` study folder (appended)

User asked to scour local files for pcaps to study, scour the internet for public reference pcaps, then create a folder and download the internet-sourced ones into it.

### Local pcap inventory (full scan of home directory)
~130+ pcap/pcapng files found (many are OneDrive-duplicate-sync paths of the same files, real distinct set is much smaller). Organized into 5 categories:
1. **Purpose-built assoc/auth/roaming reference set** (5 files, June 2) — `5320-onboarding-agent/docs/data/pcaps/`: `wpa-4way-handshake.pcap`, `wpa-eap-tls-dot1x.pcap`, `wpa2-ft-psk.pcapng`, `wpa2-ft-eap.pcapng`, `wpa3-ft-roaming.pcapng` — matches `reference_eod_20260602_wifi_auth_roaming.md`'s "5 pcap files"
2. **Real RF-compromised incident captures** — `5320-onboarding-agent/data/may7-dhcp/`, `may8-dhcp-repro/`, `may8-qos/`: May 6 GTK-rotation-wedge (guest+corporate), May 7-8 DHCP diagnosis/repro, May 8 QoS validation — each paired with a dialogue/analysis doc in the same folder
3. **Industry training captures** — `Downloads/intuitibits-wlpc-phx-2026/` (Intuitibits/WLPC: `lab11.pcapng` + filters, `intuitibits.pcapng`), `WLPC Troubleshooting/Capture 08-09` (RADIUS wireless/ethernet)
4. **Raw AP-side cloud exports** — `Downloads/cloud-*.pcap`, MAC-tagged to AH-556680 and AH-565780 (your actual lab APs); several are 0-byte empty exports, verify size before use
5. **~40 exploratory/ad-hoc captures** — lower signal, scattered across Downloads and OneDrive's two sync paths (`OneDrive-ExtremeNetworks,Inc/` and the Group Containers mirror of the same account)

### Public captures found + downloaded
- **[vanhoefm/wifi-example-captures](https://github.com/vanhoefm/wifi-example-captures)** (Mathy Vanhoef — KRACK/FragAttacks researcher) — 8 files downloaded, fills gaps the local set doesn't cover: isolated SAE handshake, EAP-PWD, WNM sleep mode, WPA3 transition mode (both bands). Passwords included in filenames/repo.
- **[wireshark/wireshark](https://github.com/wireshark/wireshark) `test/captures/`** (official Wireshark protocol-dissector test suite, current/authoritative) — 6 files downloaded: GTK rekey (`wpa1-gtk-rekey.pcapng`, directly relevant to the May 6 incident), MFP (802.11w), canonical SAE reference, **MLO/802.11be** (`wpa3-mlo.pcapng`), broad mgmt-frame decode coverage, PTK Extended Key ID
- Wireshark wiki's `SampleCaptures?action=AttachFile` URL returns an HTML wrapper, not the raw pcap — correctly avoided downloading that as if it were binary
- Not fetched, flagged for later: 2 academic papers ([Deauth Resilience Testbed](https://arxiv.org/pdf/2602.23513), [Evidence-Grounded 802.11 Diagnosis Pipeline](https://arxiv.org/pdf/2606.06871) — the second is architecturally close to the unbuilt `CorrelationAgent`/`pcap_inspector` design)

### New folder created
`/Users/khukhan/pcaps/` (new, top-level, not inside any existing project — asked-for location was ambiguous ("my directory"), defaulted to home directory and flagged the choice):
```
pcaps/
├── README.md                              ← full source attribution + per-file description + suggested study order
├── vanhoefm-wifi-example-captures/        ← 8 files
└── wireshark-official-test-captures/      ← 6 files
```
Local files (categories 1-5 above) were **not** copied/duplicated into this folder — `README.md` points to their original project paths instead, per the study-order guidance already given.

### For EOD HTML — additional item
Include: the full local pcap inventory (5 categories), the 2 public sources used with file-level descriptions, the new `/Users/khukhan/pcaps/` folder structure, and the suggested 4-step study order (local baseline → SAE/WPA3-transition gap-fill → GTK-rekey/MFP/MLO → real incident captures).

## Part 17 — Hands-on Wireshark session: finding the right capture + full narrative walkthrough (appended)

User asked to open a capture with beacon frames for direct study, then a "complex" capture showing full client lifecycle (AP turn-up → discovery → association → auth → data).

### Beacon frame review
Opened `intuitibits-single-beacon.pcapng` (WLPC/Intuitibits, purpose-built single-frame capture). Walked through IE structure: SSID, Supported Rates, DS Parameter, TIM, Country Info, RSN Information, HT/VHT/HE Capabilities. Taught display-filter application (filter bar, Expression Builder dialog, relational operators `==/!=/===/!==/>/</>=/<=/in/is present`).

### VLAN question
User asked whether VLAN assignment is visible in a beacon capture. **Answer: no** — VLAN is a wired/802.1Q Ethernet concept, not part of any 802.11 frame; tagging happens where the AP bridges wireless traffic onto the wired uplink. Two real mechanisms: static SSID→VLAN mapping, or dynamic per-user VLAN via RADIUS `Tunnel-Private-Group-ID` (attribute 81) in the Access-Accept — visible only in a wired-side capture or the RADIUS exchange itself. User already has the right files for this (`Capture 08 - RADIUS over Wireless.pcapng`, `Capture 09 - Radius over Ethernet.pcapng`), not yet opened.

### Finding a clean single-client lifecycle capture — false starts, then success
Used `tshark`/`capinfos` (found inside `/Applications/Wireshark.app/Contents/MacOS/`, not on PATH) to evaluate candidates by pulling real frame-type/timestamp data before opening anything:
- `lab11.pcapng` and `intuitibits.pcapng` (both Intuitibits/WLPC) — **rejected**: confirmed via tshark that both are scattered multi-client/multi-AP aggregates (random-looking MACs, no consistent src/dst pairing, empty/hex SSIDs) — good for frame-type variety, bad for a single coherent story
- `pcap_May7_10-40am_good.pcapng` — **rejected**: captured at Ethernet/IP layer (en0, not monitor mode), zero 802.11 headers present
- **`wpa-4way-handshake.pcap`** (local purpose-built set) — **selected**: confirmed via tshark to be one clean, coherent single-client (`00:0d:93:82:36:3a`) ↔ single-AP (`00:0c:41:82:b2:55` beacon/mgmt MAC, `00:0c:41:82:b2:53` data-plane MAC) story: Probe Req/Resp (#58-74) → Auth (#78,80) → Assoc Req/Resp (#82,84) → EAPOL 4-way (#87,89,92,94) → Data (#99+). SSID = "Coherer". This became the file used for all subsequent deep-dive work this session.

### Full narrative walkthrough delivered
Complete frame-by-frame table with real timestamps (t≈5.18s-5.66s), matched to the SANS 802.11 filter reference. User attempted to reconstruct the sequence from memory multiple times with 2 recurring errors — both corrected with evidence each time: (1) placing Association before Probe (wrong — Probe always precedes Association), (2) conflating Probe Request/Response with Authentication (two entirely separate frame-type pairs). Final correct sequence, confirmed by user: **Beacon (always on) → Probe Req/Resp → Authentication → Association Req/Resp → 4-way EAPOL → encrypted data.**

### Probe Request/Response deep dive
Confirmed via a live "Find" search screenshot: Probe Requests go to broadcast MAC, any AP in range can respond. Distinguished directed probes (named SSID, only the matching AP responds) vs wildcard probes (empty SSID, any non-suppressing AP responds — ties to the "Suppress response to broadcast probes" EN setting from Part 12). Also explained a background `CiscoLinksys_16:94:73` device probing for "linksys" — unrelated noise, every client passively probes its full remembered-network list regardless of what's actually present.

### Probe Response retry-flag finding
User's screenshot showed 9 Probe Response rows for only ~4 requests. Diagnosed via Sequence Number analysis: only 3 distinct events (SN 4031, 4032, 4036), with SN 4036 retransmitted 7 times (Retry flag set from the 2nd copy onward). **7 matches the standard 802.11 short-retry-limit convention** — likely a deliberate teaching artifact in this reference capture, though the underlying mechanism (lost ACK → retry) is identical to real RF-congestion behavior. Same 7x pattern repeated twice more later in the capture (SN 407, SN 411). Tied to the retransmission-rate health metric from the Part 15 packet-capture primer.

### Association Request/Response vs Probe Request/Response — distinguishing elements
Pulled real field content from frames #82/#84 via tshark -V:
- **Assoc Req (#82)**: Listen Interval (0x000a) — new, not in Probe; SSID; Rates; RSN IE with client's **selected** cipher (Group=TKIP, Pairwise=AES/CCMP, AKM=PSK) — contrast with Probe Response's RSN IE which only *advertises* options
- **Assoc Resp (#84)**: **Status Code = Successful (0x0000)** and **Association ID = 1** — the two elements that exist ONLY in Association Response, never in Probe Req/Resp. These are the concrete proof that Association creates real, trackable AP-side state (accept/reject + unique client handle), while Probe is entirely stateless.
- Correction made: no explicit WMM/EDCA IE was found in this specific frame pair — "class of service negotiation happens at Association" claim from Part 15 was accurate in general 802.11 terms but not confirmed present in *this* particular exchange; flagged as an honest correction rather than left unstated.

### For EOD HTML — this section
Include: the beacon-frame IE walkthrough, the VLAN-not-in-beacon explanation, the file-selection process (why 2 Intuitibits captures were rejected, why wpa-4way-handshake.pcap was chosen), the full corrected narrative table, the probe-response retry-flag forensic finding, and the Assoc-vs-Probe distinguishing-elements table (Status Code + AID as the definitive markers).

## Part 18 — Full 4-way handshake Socratic quiz + GTK deep-dive (appended)

User requested to be quizzed (Socratic method) to "get the 4-way blind." Extended interactive Q&A, all grounded in real tshark-extracted field values from frames #87/89/92/94, with corrections made at each step.

### Acronyms taught
PSK, PMK, PTK, GTK, TK, KCK, KEK, ANonce, SNonce, MIC — full definitions given, referenced throughout.

### Real field evidence pulled per message (via tshark -V)
| Msg | Key fields confirmed | 
|---|---|
| 1 (#87) | ANonce=`3e8e967d...4e6933`, Key ACK=Set, MIC=Not set (no KCK exists yet) |
| 2 (#89) | SNonce=`cdf405ce...4b1d386`, MIC=Set (`a462a702...988e45`), Key Data=22 bytes = client's RSN IE (plaintext, MIC-protected — anti-downgrade defense, re-confirming what was declared in Association Request since Association itself has no integrity protection) |
| 3 (#92) | Install=Set, ACK=Set, MIC=Set, Secure=Set, **Encrypted Key Data=Set**, 80 bytes (encrypted GTK + AP's own RSN IE), same ANonce repeated |
| 4 (#94) | MIC=Set (`10bba3bd...f2ecd1`), Nonce=all-zero, Key Data Length=0 — pure confirmation, no payload |

### Corrections made during the quiz (each with evidence)
1. **PTK is never transmitted** — independently computed by both sides once each has both nonces (client: right after Msg1, before sending Msg2, since it needs KCK to sign Msg2's MIC; AP: right after Msg2, before sending Msg3, since it needs KCK+KEK for Msg3). Not "after message 3" as user first guessed.
2. **MIC does not encrypt** — it's a signature/checksum (KCK-based), not encryption. The RSN IE in Msg2 travels in plaintext, readable directly in Wireshark — only Msg3's Key Data has the `Encrypted Key Data` flag actually set (KEK-based).
3. **GTK is not derived from PTK** — separate root key (**GMK**, generated independently by the AP), formula `GTK = KDF(GMK, AP_MAC || GNonce)` — no client MAC/nonce in it, by design (must come out identical for every client, but each client's PTK is unique). PTK's KEK sub-key is only the delivery/encryption *wrapper*, not the derivation source.
4. **GTK delivery is per-client, not one broadcast** — Msg3 exists inside each client's own individual unicast 4-way handshake; the same GTK value gets delivered N times (once per associated client), each time wrapped in that specific client's own KEK. For already-connected clients, a rotated GTK is redelivered via a separate, shorter 2-message **Group Key Handshake**, not a repeat of Message 3/the full 4-way handshake.
5. **GTK is not "continuously" created** — event-triggered (timer or client departure), producing one static value used until the next trigger, not constant regeneration.

### GTK rotation triggers (tied to user's own prior incident)
1. **Timer-based** — matches the user's own `reference_gtk_rotation_mitigations.md` "GTK update 5min" setting; shorter intervals limit exposure of a group-shared secret
2. **Client departure** — forward secrecy for group traffic, so departed clients can't keep decrypting broadcast/multicast
3. Some enterprise deployments also rotate on new-client-join (not universal)
- Direct tie flagged: the May 6 2026 GTK-rotation-cascade incident (`project_dhcp_macbook_incident.md`) is a real-world consequence of trigger #1 misbehaving — user now has the protocol foundation to revisit that root cause with more clarity

### PTK vs GTK verification via CCMP Key Index — real evidence
Pulled CCMP header fields directly from frame #102 (first unicast data frame after handshake):
- `Key Index: 0` confirmed on unicast AP→Client frame — **Key Index 0 = pairwise (PTK/TK), always, universally**
- `Key Index: 2` confirmed consistently across ALL AP-originated broadcast frames in this file (10 instances checked: #3, 26, 47, 146, 249, 337, 402, 499, 585, 631, all→`01:80:c2:00:00:00` bridge-group address) — **Key Index 1-3 = group (GTK)**, but the *specific* number (1 vs 2 vs 3) has no fixed universal meaning beyond "non-zero" — it's just whichever slot this AP's rotation state happens to be using; multiple slots exist specifically to let old/new GTK coexist during a rekey transition without dropping frames.
- **Real finding along the way**: the AP uses two different source MACs — `...b2:55` for Beacons/Probes/handshake, `...b2:53` for actual data-plane traffic (multi-radio/multi-BSS addressing offset). First filter attempt (using only `...b2:55`) came up empty on unicast data because of this — flagged as a general lesson for filtering busier real-world captures by MAC.
- Confirmed: client selects PTK vs GTK for decryption purely by reading each frame's own Key Index field — not by inferring from destination address type (though the two are always consistent in a well-formed frame).

### Why a client needs GTK despite having PTK
PTK is architecturally one-to-one (unique per client-AP pair) and cannot serve one-to-many traffic without N separate transmissions, defeating broadcast efficiency. GTK is the shared secret that lets the AP encrypt once, transmit once, and have every client decrypt the same transmission. Concrete traffic requiring GTK: ARP, DHCP broadcast/offer stages, multicast (mDNS/Bonjour, IPv6 ND).

### GTK rekey overhead in dense networks — analysis given
- Fan-out scales linearly with client count: ~2×N frames per rekey event (N clients × [delivery+ACK])
- Burst/thundering-herd effect: timer fires BSS-wide simultaneously in most implementations, concentrating key-management traffic in a short window, contending with the WMM/EDCA mechanics from Part 11
- Retry compounding: same lost-ACK/retry mechanism observed in the Probe Response forensics (Part 17) can desync a client from a new GTK in a congested channel
- Real mitigations: staggering/jittering per-client delivery; deliberately lengthening rekey interval in HD deployments (explicit security-vs-capacity tradeoff)
- **Direct tie to today's earlier work**: this is exactly the kind of second-order effect the still-undefined "dense-urban retail/mall" archetype (flagged in Part 14) would need to model — a naive capacity forecast covering only steady-state application traffic would completely miss this cost

### For EOD HTML — this section
Include: the acronym glossary, the real per-message field table, all 5 corrections made (with the specific wrong-then-right framing for each, since that's pedagogically valuable), the CCMP Key Index verification method (0=PTK/1-3=GTK) with the dual-MAC finding, the GTK-vs-PTK "why needed" explanation, and the dense-network GTK-rekey-overhead analysis tying back to the missing mall archetype.

## Part 19 — LinkedIn post review + capability correction (appended)

User asked to review a specific LinkedIn post (Harish Kumar, "Claude prompt for accuracy over helpfulness," 7-rule custom system-prompt). WebFetch **succeeded** on the LinkedIn URL, contradicting Part 9's earlier blanket claim that "LinkedIn restricts unauthenticated scraping" — that claim was too broad and is corrected here. Fetch-based summary was cross-checked against the full text the user pasted afterward and found accurate on content (engagement numbers unverifiable either way).

### Review given
**Strong point**: Rule 6 ("never invent function names/API syntax") validated with a live self-referential example from this exact session — earlier in Part 18, `eapol.keydes.msgnr` was guessed as a Wireshark field name and was wrong; the real field (`wlan_rsna_eapol.keydes.msgnr`) was only found by running `tshark -G fields` and verifying before handing it to the user. Cited as the rule "working in its strongest form" — actual verification, not verbal hedging.

**Weaknesses flagged**:
1. "Claude stops lying to you" mischaracterizes hallucination as intentional deception rather than miscalibrated confidence — a verbal instruction can only surface uncertainty already correctly assessed, it can't manufacture accurate self-assessment where none exists
2. "This works in every chat, forever" oversells a system prompt's reliability — probabilistic behavior shift, not a guarantee
3. Rule 7 (always ask clarifying questions when context is unclear), applied indiscriminately, has a real usability cost for low-stakes/reversible questions — better-calibrated version: ask when genuinely high-stakes or hard to reverse, otherwise proceed and state the assumption
4. Packaging is standard LinkedIn engagement-bait (SAVE/REPOST/Follow prompts), orthogonal to content quality but worth noting

**Synthesis given**: the stronger version of the post's advice isn't "add hedging language," it's what this session actually did throughout — when uncertain, go verify (grep the standard, run tshark, download and check real bytes) rather than just flag uncertainty verbally. Verbal hedging is the fallback for when verification isn't possible; today mostly wasn't that situation.

### For EOD HTML — this section
Include: the LinkedIn post summary + full critique (strengths/weaknesses), the self-referential `eapol.keydes.msgnr` example as proof-in-practice, and the capability correction (LinkedIn WebFetch succeeded, contradicting the earlier Part 9 blanket claim — that memory entry should be corrected).
