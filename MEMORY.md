# Memory Index

## 🔔 Active Alerts — Surface Every Session, Any Directory
- [feedback_concurrent_sessions.md](feedback_concurrent_sessions.md) — ⚠️ Up to 4 concurrent sessions active. Always re-read shared files before writing. Minimize edits to MEMORY.md + CLAUDE.md.

- **deep-research-engine** — Phase 0 done (8/8 tests, commit `4cd6b90`, May 25 2026). **Phase 1 BLOCKED** pending architecture review. Say: *"Want to do the arch review for deep-research-engine so we can start Phase 1?"* Repo: `/Users/khukhan/deep-research-engine/`. Full context in `BACKLOG.md` there.
- **802.11 Mastery curriculum** — Phase 1 Day 1 NOT STARTED (last: 2026-05-28). OFDM question loaded. Say *"start curriculum"* to begin. Context: `project_wifi_mastery_progress.md`.
- **Sprint C EP1 Traverser** — BLOCKED: xcloud bins blocked (`ws-routed-parcel` won't mount). Say *"resume Sprint C"* to continue. Repo: `xiq-ep1-intelligence-engine/`. Context: `memory/active_session_resume.md`.
- **Multi-LLM research framework** — plan approved 2026-05-24, code not started. Greenfield at `/Users/khukhan/multi-llm-research/`. Say: *"start multi-LLM Phase 0"* to begin.
- **May 24 EOD** — update `/Users/khukhan/docs/session_summary_20260524.html` to add 11 Socratic Agent HTML files (wifi-80211-mastery: 6 phases, references.html, assessments.html, 76 Qs).
- **Index dedup** — `session_summary_20260422.html` linked twice on all 3 index pages of 5320-onboarding (banner button + EOD card). Consolidate to one pointer.
- **Architect_CodeAgent (May 28)** — EODs pushed (52df80e, d3cb614), two-agent system live, unified EOD template pending (merge May 24 sticky TOC + May 25 protocol flows). Say: *"continue architect session"* to pick up.
- **FIFA WC Simulator** — Sprint 1 COMPLETE (v3). Sprint 2 PLANNED (18 items). Say *"start Sprint 2"* to begin. Plan: `project_simulator_sprint2_plan.md`.
- **DigitalTwinEngine** — Sprint 2a+2e COMPLETE. NEXT: pcap test → Sprint 2b (knob_panel.py). Disable ADSP in EP1 first. Context: `project_pcap_sprint2e_architecture.md`.
- **XE Socratic bank** — 9/50 complete (last: XE-08,11/12/15/16/17/19/20/21). Next: continue bank or build `scenario_bank_realworld_june11.html` (77 tickets).
- **AP1 EP1 actions PENDING**: disable ADSP, raise basic rate 12Mbps, PMK caching 86400s, roaming hops 2, QF_Modem DHCP 24hr. wifi0 admin disabled — re-enable via EP1 radio profile (NOT CLI).
- **AP2 (AH-565780) ONBOARDED June 16** — CAPWAP RUN, both SSIDs active, port 5 tagged. Client assoc RESOLVED (ghost device AH-901340 removed). EOD: `docs/session_summary_20260616_ap2_onboard.html` commit 51b6ce7.
- **RADIUS sprint** — DEFERRED post-PTO. AP2 in Karl's VIQ. Auth rejected — creds not in Karl's RADIUS DB. Next: Karl adds user, re-run `_debug auth basic`. Context: `project_session_20260630_radius_vlan.md`.
- **WiFi 7 Digital Twin Sprint** — Phase 0 never started (June 17 2026 deadline passed). Repo `/Users/khukhan/wifi-7-twin/` doesn't exist yet. Full plan: `project_wifi7_twin_sprint.md`.

> To clear an alert: delete its line once the action is complete.

---

## Archive
- [memory_archive_dormant.md](memory_archive_dormant.md) — All stale/completed/dormant entries moved from MEMORY.md to stay under size limit. EOD refs (May 24/25/28/Jun2), completed sprints, old session agendas, future-sprint placeholders. Search here before concluding context is lost.

---

## References
- [reference_backlog_system.md](reference_backlog_system.md) — BACKLOG.md + CLAUDE.md session-start pattern: every project gets a backlog; Claude reads it first and briefs 3–5 bullets. First instance: 5320-onboarding-agent/BACKLOG.md (2026-05-24).
- [reference_wlpc_corpus.md](reference_wlpc_corpus.md) — 150+ private WiFi files at /Users/khukhan/Downloads/WLPC Troubleshooting — tiered for RAG (T1: 802.11be-2024, Rev Wi-Fi, WLANPros; T2: Aruba VHD, Apple; T3: WLANPi, Wireshark). NEVER on GitHub.
- [reference_wlanpros_checklists_v5.md](reference_wlanpros_checklists_v5.md) — All 8 sheets of WLANPros Checklists v5.xlsx: Top 20, Extended (items 21-79), Connection test, Not-Wireless pre/post, Passpoint, WiFi 7 MLO, eduroam, Apple Best Practices
- [reference_wlpc_2026_troubleshooting.md](reference_wlpc_2026_troubleshooting.md) — WLPC Express 2026: 30-row potential causes table (WIRELESS/LOCAL NET/INTERNET), 6-process framework (MANAGE/INVESTIGATE/MEASURE/VALIDATE/ANALYZE/INTERROGATE), frame stats
- [reference_revolution_wifi_capacity_planner.md](reference_revolution_wifi_capacity_planner.md) — Complete Rev Wi-Fi Capacity Planner methodology (pp.1-38): airtime formula, all Network Design inputs, Capacity Demand rows, RF Coverage Design RSSI tiers, Capacity Results outputs, SSID Overhead Calculator. Private.
- [reference_voss_cli.md](reference_voss_cli.md) — VOSS/FabricEngine 9.3.x confirmed CLI: navigation, IP, routes, VLAN, IQAgent, show commands, boot warnings, factory reset, VOSS vs EXOS diff table.
- [reference_exos_cli_syntax.md](reference_exos_cli_syntax.md) — EXOS SwitchEngine 33.x verified CLI syntax: DHCP server (dhcp-address-range + dash separator + port required), routing, gotchas
- [reference_claude_best_practice.md](reference_claude_best_practice.md) — Claude Code best practice reference repo (github.com/shanraisshan/claude-code-best-practice)
- [reference_global_config.md](reference_global_config.md) — Global Claude Code config repo (github.com/khursheedkhanaiforgood-ai/claude-global-config) — synthesized 16 sources 2026-04-06, ALL projects
- [reference_github_cisco_agent.md](reference_github_cisco_agent.md) — GitHub PAT location for cisco-en-cli-agent repo (token in project .env, remote URL pre-configured)
- [reference_pg_dump_backup.md](reference_pg_dump_backup.md) — pg_dump/pg_restore full commands, Railway PostgreSQL backup location, known backup log
- [reference_5320_pages_source.md](reference_5320_pages_source.md) — 5320-onboarding GitHub Pages serves from MAIN (not gh-pages despite branch existing). EODs must land on main + update 3 index pages (index.html, index-nyt.html, index-harpers.html) for parity.
- [reference_exos_voss_4_principles.md](reference_exos_voss_4_principles.md) — EXOS→VOSS Rosetta Stone: 4 principles (Brain=ISIS, Muscle=SPB, Service=I-SID, Edge=Auto-Sense/FA). Mental hooks for every translation step. Pair w/ reference_voss_cli.md.
- [reference_dhcp_wifi_triage_runbook.md](reference_dhcp_wifi_triage_runbook.md) — 5-stage triage flow for "Wi-Fi client can't get DHCP" across AP / XIQ / EXOS. Capture-before-reboot rule. Smoking-gun decrypt-counter procedure. Distilled from May 6 incident.
- [reference_gtk_rotation_mitigations.md](reference_gtk_rotation_mitigations.md) — Layered mitigations for May 6/8 GTK rotation cascade on AP3000 + WPA3-SAE-PMF. Top 3: DHCP lease 24h, 802.11r FT, GTK update 5min. For GTAC/R&D escalation.
- [reference_ap3000_command_themes.md](reference_ap3000_command_themes.md) — AP3000 (IQ Engine OS) commands in 4 lenses: Design/Configure/Optimize/Troubleshoot. Includes show + debug + kdebug + capture + frame_filter syntax (DHCP, decrypt-error filters). Source: AP1 SSH dump May 7.
- [reference_exos_command_themes.md](reference_exos_command_themes.md) — EXOS SwitchEngine 33.x commands in same 4-lens structure. VLANs, FDB, DHCP, ACL, mirroring, ports stats. Source: SW1 tech-support May 7.
- [reference_voss_command_themes.md](reference_voss_command_themes.md) — VOSS FabricEngine 9.3.x commands in same 4-lens structure. Adds fabric layer: IS-IS, SPB, I-SID, FA. Maps to the 4-principle Rosetta Stone. Source: SW2 onboarding context.
- [reference_voss_script_bank.md](reference_voss_script_bank.md) — VOSS CLI script bank built through D1-D9 Socratic sessions (Aug 2026). D1: backbone/IS-IS/B-FIB. D2: I-SID binding + Anycast Gateway. D3-D9 TBD.
- **EOD Journal repo:** `/Users/khukhan/lab-journal/` — git-initialized on main. Push to GitHub (private) this week. EOD files: `session_summary_YYYYMMDD.html`.
- [reference_ep1_cloud_architecture.md](reference_ep1_cloud_architecture.md) — EP1 Cloud Architecture: microservices+Kubernetes, GDC=AWS-only/RDC=multi-cloud, AI Core 3-component (Orchestration+Services+DataHub), HITL=default+RBAC-scoped, EP1 not in data path.
- [reference_ep1_extreme_platform_one.md](reference_ep1_extreme_platform_one.md) — EP1 = Extreme Platform One (next-gen XIQ portal, 9-dot launcher). NOT the IPE SD-WAN device. When EP1 is down: deploys blocked, PPSK RadSec affected, check cloud status page.
- [reference_voss_standards_rfcs.md](reference_voss_standards_rfcs.md) — 5 IEEE/IETF standards behind VOSS: 802.1aq (SPB), RFC 6329 (IS-IS-for-SPB), 802.1ah (PBB MAC-in-MAC), 802.1Qcj (FA), 802.1ag (CFM). Maps each to the 4-principle Rosetta Stone.
- [security_assessment_cisco_en_cli_agent.md](security_assessment_cisco_en_cli_agent.md) — Tier-1 threat assessment (NIST 800-53 + CISA): 3 Critical, 7 High, 5 Medium, 2 Low. Immediate: revoke GitHub PAT + rotate passwords + fix SQL injection + re-enable CORS/XSRF
- [reference_new_repos_20260408.md](reference_new_repos_20260408.md) — claude-mem (thedotmack), Boris Cherny X post hidden features, Taggart critical perspective, LinkedIn Claude Code tips April 2026
- [reference_claude_patterns_may2026.md](reference_claude_patterns_may2026.md) — Actionable Claude Code patterns May 2026: Hooks vs CLAUDE.md, /compact directed, Writer/Reviewer, fan-out loop, FINDINGS.md, Karpathy KB production warnings, Boris Cherny hidden features quick-ref

## Feedback
- [feedback_memory_cleanup_ask_first.md](feedback_memory_cleanup_ask_first.md) — ALWAYS ask before removing/trimming/archiving any memory entry. No silent deletions. Confirmed 2026-08-05.
- [feedback_no_hallucination_cli.md](feedback_no_hallucination_cli.md) — NEVER guess CLI commands. If unsure, say so and ask permission to experiment. Applies to all CLI environments.
- [feedback_best_practices.md](feedback_best_practices.md) — Always auto-apply Claude Code Best Practices to every new project repo
- [feedback_main_branch_protection.md](feedback_main_branch_protection.md) — eve-ng-lab-platform main branch is locked; no pushes without user password authorization
- [feedback_macos_wpa2_ssid_hiding.md](feedback_macos_wpa2_ssid_hiding.md) — macOS Ventura+ hides WPA2-only SSIDs and grays out Join. Fix: WPA2/WPA3 transition mode in XIQ. iPhone unaffected.
- [feedback_ppsk_manual_entry.md](feedback_ppsk_manual_entry.md) — Always type PPSK manually — never paste from email/SMS. Invisible chars cause silent Join failure. Use simple test password to eliminate variable.
- [feedback_xiq_supplemental_cli.md](feedback_xiq_supplemental_cli.md) — ${vlan:NAME} resolves to VLAN ID not name — EXOS rejects it. Always hardcode VLAN name. XIQ drops underscores (Guest_100 → Guest100). Verify with show vlan first.
- [feedback_multi_session_writes.md](feedback_multi_session_writes.md) — 4 concurrent sessions share MEMORY.md, CLAUDE.md, project_wifi_mastery_progress.md. Always Read before Edit on any shared file — never edit from cached context.
- [feedback_session_startup_briefing.md](feedback_session_startup_briefing.md) — On every login: read Active Alerts, brief user on all session states before doing anything else. Format: bullet list per session with status + next action.

## Projects
- [project_wifi7_twin_sprint.md](project_wifi7_twin_sprint.md) — WiFi 7 Digital Twin sprint: Phase 0 starts June 17. Omniverse→Sionna→ns3-Sionna→Streamlit. Repo: /Users/khukhan/wifi-7-twin/. Docker CPU MacBook.
- [project_wifi_80211_mastery_repo.md](project_wifi_80211_mastery_repo.md) — Public repo May 25: wifi-80211-mastery. 11 HTML files, 6 phases (34 days), assessments.html (46 Qs + 30-Q CWNA mock). Live on Pages.
- [project_ppsk_dot1x_sprint.md](project_ppsk_dot1x_sprint.md) — PPSK self-reg gap (no domain filter in EP1 v25.9.0). Toggle: "Return Aerohive Private PSK". Future sprint: 802.1X+RADIUS/NPS when available.
- [project_voss_fabric_migration.md](project_voss_fabric_migration.md) — EXOS→VOSS migration: voss_migration_horizon.html LIVE (35 sections). Pending: add RFC/IEEE hyperlinks to D1.
- [project_5320_onboarding.md](project_5320_onboarding.md) — May 15: SW1 PPSK VLAN split CONFIRMED (Staff→VLAN1, Students→VLAN10). 802.1X/NPS deferred. Full config + backlog in file.
- [project_wifi_digital_twin.md](project_wifi_digital_twin.md) — Full Digital Twin architecture: 3 operational loops, complete agent topology, RRM Simulation + EP1 + Tuning Agent, Broadcom chipset param map, 5G↔WiFi bridge, Sprint 1-3 build sequence
- [project_digital_twin_calibration_theory.md](project_digital_twin_calibration_theory.md) — Calibration theory June 9: two-source architecture, capMult accuracy, 5 archetypes, 20-source RAG library, pre-Sprint 3 action items
- [project_pcap_sprint2e_architecture.md](project_pcap_sprint2e_architecture.md) — PacketCapture architecture June 10: Sprint 2e done, Sprint 2f (pcap_library) design, CorrelationAgent on-demand design, Wireshark 3-moment timing, per-session DB frozen
- [project_wifi_dpm.md](project_wifi_dpm.md) — Design Parameter Manifest: ~60 params in 7 sections (A=Site, B=Clients, C=Apps, D=Network, E=Channel, F=Security, G=RRM). Socratic → DPM review → calibrate → engine runs. Three source flags.
- [project_wifi_digital_twin_sprint1.md](project_wifi_digital_twin_sprint1.md) — Sprint 1 plan READY, coding NOT STARTED. Next session = Day 1: DPM models + Link Budget Engine + Airtime Calculator. 17 modules in 6 batches total.
- [project_xiq_ep1_transition.md](project_xiq_ep1_transition.md) — 6-week sprint May 15–Jul 1: exhaustively map every XIQ feature to EP1 for client transition guide. XIQ GUI retired Jul 1. Branch: feature/xiq-ep1-transition.
- [project_ep1_transition_resources.md](project_ep1_transition_resources.md) — Full catalog of EP1/XIQ migration PDFs at /Users/khukhan/Downloads/EP1 and Other Items for Migration/ (3 folders). #engine section needs live links.
- [project_intelligence_engine_architecture.md](project_intelligence_engine_architecture.md) — May 21: 17-agent LangGraph, 3-layer confidence, Sprint A done (25 tests green). Sprint B Playwright scrape was Jul 1 deadline.
- [project_ppsk_study_may18.md](project_ppsk_study_may18.md) — May 18-19: PPSK guide read, ep1-deployment-guide.html v2.7 (fe227f6), xiq-ep1-arch-debate.html. 7 open R&D questions in Section 9.
- **PPSK Sprint 3 guide:** `~/OneDrive-ExtremeNetworks/Private-PreShared-Key-PPSK-Guide_v6 2 1.pdf` — KK-annotated v6.2.1. Use for Sprint 3 PPSK dynamic VLAN lab.
- [project_deep_research_engine_phase0.md](project_deep_research_engine_phase0.md) — deep-research-engine Phase 0 COMPLETE (2026-05-25, commit 4cd6b90). LangGraph+LiteLLM+Tri-Agent. **May 26: revisit architecture before Phase 1.**
- [project_digital_twin_live_ap_calibration.md](project_digital_twin_live_ap_calibration.md) — DigitalTwinEngine architecture June 5 2026: 6-agent topology, 15-param AP→Simulator bridge, FederatedTwinAgent MC-gradient calibration loop, Sprint 2/3 plan.
- [project_track_b_calibration.md](project_track_b_calibration.md) — Track B ns-3 calibration: 4-level validation sequence, surrogate calibration loop, 60-param classification, 12→15 search knob promotion, ns-3 fabricated API warning.
- [project_wifi_mastery_progress.md](project_wifi_mastery_progress.md) — ⬅ READ THIS FIRST on login. Phase 1 Day 1 NOT STARTED (last session 2026-05-28). OFDM question loaded. Unified EOD template + multi-LLM framework pending.
- [project_simulator_v2_bugs.md](project_simulator_v2_bugs.md) — 17 physics bugs in stadium_wifi_simulator_v2.html (June 3). C1/C2/C3 critical: capacity 46,000 Gbps (should be ~2,000), airtime 2% (should be 20–60%). Sprint 1-4 plan.
- [project_two_agent_collaboration.md](project_two_agent_collaboration.md) — Two-agent setup: Architect_CodeAgent (me) + Socratic_CurriculumAgent. Shared state file, two threads (curriculum + multi-LLM framework), daily routine, kickoff prompt.
- [project_session_20260630_radius_vlan.md](project_session_20260630_radius_vlan.md) — June 30: AP2→Karl's VIQ CAPWAP debug, RADIUS deferred post-PTO, tagged/untagged deep-dive, Sacramento remote passive capture strategy (3 options).
- [project_sprint_c_ep1_traverser.md](project_sprint_c_ep1_traverser.md) — Sprint C scraper: extractor+traverser DONE but xcloud bins blocked (ws-routed-parcel won't mount, org context not persisting). Next: intercept XHR on org click.
- [project_kb_school_sprint.md](project_kb_school_sprint.md) — KB_School lab (May 14-15): PPSK VLAN split, Supplemental CLI lessons. Key fix: `enable ipforwarding vlan Default` after every factory reset.
- [project_session_20260713_dpm_knob_distillation.md](project_session_20260713_dpm_knob_distillation.md) — July 13 (Parts 1-19): file audit, cisco-en-cli-agent Critical issues still open, 64-param DPM + 12-knob distillation, 3-band EN radio matrix, Cisco RRM vs AutoRF analysis, Wireshark/4-way/GTK Socratic. EOD LIVE commit 92ec4a3.

## Feedback (continued)
- [feedback_ap_reboot_xiq.md](feedback_ap_reboot_xiq.md) — AP data plane corruption: fix = XIQ-triggered reboot (Monitor→Devices→AP→Reboot), NOT power cycle. Broadcasts work, unicast breaks silently.
- [feedback_wired_wireless_diagnostic.md](feedback_wired_wireless_diagnostic.md) — 4-step chain: show fdb → show iparp → SW1 ping client → `arp -an` on MacBook (NOT arp -n). Empty cache = smoking gun.
- [feedback_test_before_commit.md](feedback_test_before_commit.md) — Test deployment configs before pushing; don't commit guesses that fail in Railway/CI
- [feedback_exos_factory_reset.md](feedback_exos_factory_reset.md) — EXOS factory reset = `unconfigure switch all` (NOT `delete /intflash/config.cfg` — Fabric Engine only)
- [feedback_voss_iqagent_dns.md](feedback_voss_iqagent_dns.md) — VOSS: ip name-server does NOT feed IQAgent DNS. Must use DHCP at boot (ZTP+) for /etc/resolv.conf. Static IP breaks XIQ cloud agent.
- [feedback_eod_html_format.md](feedback_eod_html_format.md) — EOD HTML = NYT journal style (white, Libre Baskerville serif, adaptive TOC). Dark theme only for pure reference pages.
- [feedback_fdb_vlan_verification.md](feedback_fdb_vlan_verification.md) — `show fdb ports X` is authoritative VLAN truth; ARP ages 20 min. Dual-MAC in two ARP tables = expected when testing both PPSK passphrases on same device.
- [feedback_supplemental_cli_idempotency.md](feedback_supplemental_cli_idempotency.md) — `enable dhcp` and `enable ipforwarding` are NOT idempotent on EXOS — hang XIQ at 15%. Run manually post-factory-reset only.
- [feedback_ssh_hostkey_reset.md](feedback_ssh_hostkey_reset.md) — After AP factory reset: SSH blocks with "REMOTE HOST IDENTIFICATION HAS CHANGED". Fix: `ssh-keygen -R <ip>` then reconnect.
- [feedback_iqagent_heartbeat.md](feedback_iqagent_heartbeat.md) — "proxy device-connector unknown POST /health-check/[serial]" at ~60s = normal IQAgent keepalive, NOT an error.
- [feedback_ep1_stale_cache_mac_random.md](feedback_ep1_stale_cache_mac_random.md) — show station on AP is authoritative; EP1 client view lags. macOS randomizes MAC per SSID — verify with networksetup -getmacaddress en0.
- [feedback_copyright.md](feedback_copyright.md) — Add copyright insignia to all HTML pages (5320-onboarding + engine). Flagged May 21, deferred.
- [feedback_curriculum_agent_write_boundary.md](feedback_curriculum_agent_write_boundary.md) — Socratic Curriculum Agent must only write to project_wifi_mastery_progress.md — never to MEMORY.md (Architect_CodeAgent domain only).
- [feedback_simulator_font_audit.md](feedback_simulator_font_audit.md) — Font audit for simulator v2 deferred to Sprint 2. All 8-9px overrides → 11px. Bundle with Sprint 2 structural changes after physics fix.
- [feedback_validate_model_before_features.md](feedback_validate_model_before_features.md) — Validate simulator physics (airtime 20-60%, capacity ~2,000 Gbps) before MC/optimizer. June 3: model was off 20× — C1/C2/C3 critical.
- [feedback_memory_overwrite.md](feedback_memory_overwrite.md) — Never silently overwrite an existing memory file. Always show old vs new, offer keep/restore/review. Triggered 2026-05-25.
- [reference_5320_repo_pages_config.md](reference_5320_repo_pages_config.md) — Detailed 5320-onboarding Pages config: main branch root, 3-index update convention, index card border colour ledger (Apr20–May7). More detail than reference_5320_pages_source.md.
