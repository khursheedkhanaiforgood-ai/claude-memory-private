---
name: Dormant / Archived Memory Entries
description: Stale, completed, or dormant project/reference/feedback entries moved from MEMORY.md to keep the index under size limits. All data preserved here. Search this file to find context for any old sprint, session, or project.
type: reference
originSessionId: 3db2d693-1e37-4eb5-8f10-fc421a5db74d
---
# Archived Entries — Moved from MEMORY.md

## Entries removed during Aug 5 2026 MEMORY.md cleanup

### EOD Reference Pointers (HTML session summaries)
- [reference_eod_20260524.md](reference_eod_20260524.md) — EOD HTML for 2026-05-24 session at /Users/khukhan/docs/session_summary_20260524.html — covers two-agent architecture + multi-LLM framework plan.
- [reference_eod_20260525.md](reference_eod_20260525.md) — EOD HTML for 2026-05-25 at /Users/khukhan/deep-research-engine/docs/session_summary_20260525.html — Phase 0 scaffold done; May 26 = architecture review before Phase 1.
- [reference_eod_20260528.md](reference_eod_20260528.md) — EOD HTML for 2026-05-28 at /Users/khukhan/deep-research-engine/docs/session_summary_20260528.html — memory fix, arch review still pending.
- [reference_eod_20260602_wifi_auth_roaming.md](reference_eod_20260602_wifi_auth_roaming.md) — June 2 study EOD: 802.11 Auth/Assoc/Roaming — PSK/EAP-PEAP/EAP-TLS/4-way/802.11r FT + 5 pcap files. Live: 5320-onboarding/docs/session_summary_20260602_wifi_auth_roaming.html

### Completed / Superseded Projects
- [project_training_deadline.md](project_training_deadline.md) — All 6 sessions COMPLETE Apr 22; 5/5 criteria PASS; Socratic log live; index parity done; team review Apr 23.
- [project_multi_llm_research_framework.md](project_multi_llm_research_framework.md) — Earlier draft plan (2026-05-24): FastAPI+ChromaDB+Streamlit stack, GPT-4o, /Users/khukhan/multi-llm-research/ (superseded by deep-research-engine below)

### Feedback (less-detailed duplicate — kept detailed version in MEMORY.md)
- [feedback_memory_overwrite.md](feedback_memory_overwrite.md) — Never silently overwrite a memory file. Always restore original + create new file alongside, then offer user the choice. (Shorter version — see detailed entry in active MEMORY.md)

---

## Option A stale entries — dormant sprints / future placeholders

### EVE-NG / CLI Agent (dormant since April 2026)
- [project_5320_session_20260408.md](project_5320_session_20260408.md) — April 8 morning: timed 30min E2E XIQ+CLI deployment walkthrough
- [project_voss_simulator.md](project_voss_simulator.md) — Digital Twin Simulator (Apr 8 2026): SOA Python, 7 services, 18-step FSM, 16 tests. Run: python3 -m simulator. Next: agentic E2E + Railway
- [project_exos_layer2_lab.md](project_exos_layer2_lab.md) — EXOS Layer 2 ring lab HTML (EVE-NG, 3 tabs, #me session log). Pending: Tab 4 to make session log a visible public tab
- [project_branch_freeze.md](project_branch_freeze.md) — feature/traceroute is FROZEN (tested, stable); active work on feature/windows-widget (auto-start widget)
- [project_cisco_en_cli_agent.md](project_cisco_en_cli_agent.md) — CISCO-EN CLI Mapping Agent: Sprint 14 full day 2026-04-03 (16 commits). Confidence calculator, input callouts, print CSS, session logging, clean 2-col view. Pending: repetition fix + merge to main

### Future Sprints (no date, no immediate action)
- [project_user_management_sprint.md](project_user_management_sprint.md) — Future sprint: DB-backed users, first-time pw change, logout UX fix, admin UI. Current workaround: edit APP_USERS JSON in Railway env vars.
- [project_concurrency_sprint.md](project_concurrency_sprint.md) — Future sprint: fix Streamlit session blocking during Claude API calls. Option 1: streaming (1 day). Option 2: background threading (3 days, fully unblocks tabs).
- [project_sprint16_reverse_translation.md](project_sprint16_reverse_translation.md) — Sprint 16: EXOS→Cisco IOS/IOS-XE reverse translation. Needs new system prompt + EXOS parser. DB already bidirectional. Est. 3-4 days.

### Old Session Review Agendas
- [project_review_20260526.md](project_review_20260526.md) — May 26 review agenda: 4 items (verify repo URL, update May 24 EOD, add write-boundary feedback, fix index dedup). Curriculum Phase 1 Day 1 queued. Multi-LLM framework Phase 0 ready.

---

## Aug 5 2026 — Second archive pass (Projects section cleanup)

### Historical Session Notes (no pending actions)
- [project_session_20260611_ap_cli.md](project_session_20260611_ap_cli.md) — June 11 AP CLI learnings: terminal length invalid on IQ Engine, _debug vs _kdebug, logfile /tmp redirect, show station for client VLAN.
- [project_session_20260520_ep1_sprint.md](project_session_20260520_ep1_sprint.md) — May 20: EP1 zero-CLI deploy, PPSK VLAN5 + Guest VLAN10, GuestEssentials + AirDefense. Full CLI + EP1 steps in file.
- [project_session_20260521_ep1_deploy.md](project_session_20260521_ep1_deploy.md) — May 21 Stage 2 EP1 zero-CLI deploy sprint. SW1 factory reset complete. AP1 claimed+pending factory reset. G-01 gap logged. Next: claim SW1, build Switch Template.
- [project_session_20260507.md](project_session_20260507.md) — May 7 sequence: gap fill DONE → DHCP DIAGNOSED + EOD live + linked from landing page (commit dcccb27) → next: EXOS→VOSS 4-principle cheatsheet + SW2 config + WiFi DT Sprint 1.
- [project_session_20260508.md](project_session_20260508.md) — May 8 DHCP repro DONE. Hypothesis B confirmed via 3-client fingerprint. Trigger refined: en8→Port 5 dual-interface event. Self-recovery 2-4 min. EOD live.
- [project_session_20260511.md](project_session_20260511.md) — May 11: EXOS→VOSS 4 principles on SW2 live config. 802.1X PEAP-MSCHAPv2 arc. dot1x_simulator.html COMPLETE. 802.1X/NPS deferred.
- [project_notebookllm_sprint_b_may22.md](project_notebookllm_sprint_b_may22.md) — NotebookLLM May 22: 7 scli_safety_rules (G-01 to G-07) for Node 0 Path Planner; Node 11 cross-val agent prompt + schema. Both JSON files created in sprint_b/.

### Completed Sprints / Resolved Incidents
- [project_karl_lab_apr21.md](project_karl_lab_apr21.md) — Apr 28 EOD: debrief_apr21.html COMPLETE — all 15 slides CLI config blocks; Pure L2 Update deck (12 slides). Pending: verification checklist, PSK key value, OWE toggle.
- [project_guest_onboarding_sprint.md](project_guest_onboarding_sprint.md) — Phase 1 ✅. Firewall restored (192.168.x blocked). CWP self-reg parked. CAPWAP RFCs: 5415/5416/5417/6347. Walk-in SSID = Guest_Wireless VLAN 30.
- [project_eapol_forensics_may4.md](project_eapol_forensics_may4.md) — May 4: 306K-frame Wireshark forensics, WPA2/WPA3 key hierarchy, rogue NAV attack (e0:3e:44:10:aa:64). Q2 resolved: RDC tunnel = GRE.
- [project_dhcp_macbook_incident.md](project_dhcp_macbook_incident.md) — May 6 lab incident DIAGNOSED: WPA2 key-state wedge (TK desync MacBook + GTK rotation iPhone). Validated by before/after pcap on same Guest LAA MAC.
- [project_qos_egress_validation.md](project_qos_egress_validation.md) — May 8 PM QoS bug validated. 3 findings: (1) weights all=1 FIXED, (2) VLAN qosprofile ingress-only architectural, (3) DSCP=0 end-to-end. ACL fix pending.
- [project_index_dedup_pending.md](project_index_dedup_pending.md) — PENDING: session_summary_20260422.html linked twice on all 3 index pages. Consolidate to one pointer. (Also tracked in Active Alerts.)
- [project_kb_school_sprint.md](project_kb_school_sprint.md) — Wait — kept in MEMORY.md (key CLI fix).

### Superseded / Stale Work
- [project_xiq_switch_policy_vlan100.md](project_xiq_switch_policy_vlan100.md) — Khursheed_SW1_VLAN100 policy: SW1 factory reset done, Supplemental CLI STUCK (Guest_100 vs Guest100). Stale "resume morning" from months ago.
- [project_boss_viq_capwap.md](project_boss_viq_capwap.md) — Boss Karl's VIQ: provision AP via CAPWAP (RFC 5415/5416). Access granted May 13. Superseded by AP2 onboarding June 16.
- [project_phase4a_xiq_wizard.md](project_phase4a_xiq_wizard.md) — Phase 4a: XIQ intent-based wizard (manager approval needed). NLP parse → deterministic XIQ API gen + RAG. Deferred indefinitely.
- [project_meridian_retail.md](project_meridian_retail.md) — Meridian Retail Group: SEPARATE engagement from Horizon. Apr 23 consultant exercise. Never conflate with Horizon/Karl Lab.
- [project_5320_new_arch_voss_ipe.md](project_5320_new_arch_voss_ipe.md) — May 1: IPE live, VIQ onboarded SW2. VLAN 4047 SD-WAN / VLAN 4048 onboarding. Corp VLAN50/I-SID 100050, Guest VLAN60/I-SID 100060.
- [project_voss_simulator.md](project_voss_simulator.md) — Digital Twin Simulator (Apr 8 2026): SOA Python, 7 services, 18-step FSM, 16 tests. Next: agentic E2E + Railway. (Dormant — also in Option A.)
- [project_cricket_guide.md](project_cricket_guide.md) — Cricket Guide app (CLI + Streamlit GUI), deployed to GitHub + Streamlit Cloud. GUI needs debugging.
- [project_eve_ng_pending.md](project_eve_ng_pending.md) — EVE-NG Lab Platform: two deferred features (NSSM Windows service, EVE-NG folder browser + topology).
- [project_exos_layer2_lab.md](project_exos_layer2_lab.md) — EXOS Layer 2 ring lab HTML (EVE-NG, 3 tabs, #me session log). Pending: Tab 4. (Also in Option A.)

### Design Docs / Completed Architecture (context lives in active project files)
- [project_voss_simulator_sprint_plan.md](project_voss_simulator_sprint_plan.md) — VOSS Simulator Sprint 1+2 plan: 7 themes/bins, ExplainService, RAG corpus plan, topology.
- [project_wifi_mastery.md](project_wifi_mastery.md) — Site structure Apr 29: NYT landing = root index.html; dark KB = docs/wifi-mastery-guide.html; docs/ has HLD/LLD/design-journey/EOD.
- [project_wifi_branch_strategy.md](project_wifi_branch_strategy.md) — Two-branch strategy: feature/wifi6-baseline (MCS 0-11) + feature/wifi7-complete (MCS 0-13, MLO, 320MHz). Mixed-mode deferred Sprint 4+.
- [project_wifi_lld_phase1.md](project_wifi_lld_phase1.md) — LLD Phase 1 COMPLETE ✓ (Apr 29). 9 modules fully spec'd + Appendix A. HTML live at GitHub Pages.
- [project_xiq_ep1_transition.md](project_xiq_ep1_transition.md) — 6-week sprint May 15–Jul 1: exhaustively map every XIQ feature to EP1 for client transition guide. XIQ GUI retired Jul 1. Branch: feature/xiq-ep1-transition.
- [project_xiq_ep1_architecture.md](project_xiq_ep1_architecture.md) — Full architecture design dialogue May 15: hypothesis, 25 thematic bins, two-agent RAG scrape results, front-end design pattern (Cisco→EXOS template), context engine spec.
- [project_ep1_transition_resources.md](project_ep1_transition_resources.md) — Full catalog of EP1/XIQ migration PDFs at /Users/khukhan/Downloads/EP1 and Other Items for Migration/ (3 folders). #engine section needs live links.
- [project_engine_model_debate_socratic.md](project_engine_model_debate_socratic.md) — "Tomorrow Socratic" (May 2026): non-Anthropic model evaluation + multi-LLM red-team of engine architecture.
- [project_engine_sprint_e_ui_modes.md](project_engine_sprint_e_ui_modes.md) — Sprint E: two query modes (fast one-shot + guided HITL). Default = fast; HITL = button. Deferred May 21.
- [project_wifi_mastery_curriculum_review.md](project_wifi_mastery_curriculum_review.md) — 802.11 Mastery Curriculum reviewed 2026-05-24: 6-phase 34-day Socratic plan. 5 improvement suggestions deferred.
- [project_v3_evaluation_dialogue.md](project_v3_evaluation_dialogue.md) — Verbatim June 4 v3 evaluation: 20+ UI/physics/MC/optimizer questions, 15-item issue log, archetype system decision.
- [project_v3_sprint1_postscript.md](project_v3_sprint1_postscript.md) — Sprint 1 fix table: C1/C2/C3/L1 equations, reasoning, before/after metrics.
- [project_simulator_agentic_framework.md](project_simulator_agentic_framework.md) — Agentic framework: LangGraph + 7 sub-agents (Physics/Parameter/MC/Optimizer/Scenario/Calibration/UI/Report). Stack: LangGraph+Claude SDK+NumPy.
