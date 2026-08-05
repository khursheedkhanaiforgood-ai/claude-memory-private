---
name: Simulator v3 Sprint 2 Dialogue — June 4 2026
description: Verbatim user dialogue on archetypes, DL/UL AP concurrency, ns-3 time-series, and Sprint 2 planning. For EOD HTML.
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---

## VERBATIM USER DIALOGUE (keep for EOD HTML)

"We need to evaluate the 'naming of the archetypes from the Claude deliberations yesterday. Pls review. We need to give the option to set the SLA parameters for each archetype to the user in a boundded range or give them a 'standard option to begin with'. We need to also give the user the option to select number of APs. The question I hav enow is, how do we 'simulate the concurrency of the number of AP's on the downlink? Or uplink? After this iteration, pls work with me to fix the issues we identified above. HOLD ALL THIS DIALOGUE IN MEMORY verbatim so we can link-list at EOD for the way we debugged and upgraded. My question is, bigger... on the ns3 integration, how can we 'run this simulation against time' and 'see the peformance of the system' as a function of it, where the 'user can define the load changes a priori' or as we progress... Keep that for the next phase of v3 sprint. But for now, do you know what I am looking for, pls clarify? /plan too."

## KEY DECISIONS FROM THIS DIALOGUE

1. Archetype system: selectable venue type, each with its own frozen/search param split + SLA targets
2. SLA parameters: user-editable within bounded range, with "Standard" preset per archetype
3. AP Count: already added (L1), but archetype should set a sensible default per venue
4. DL/UL AP concurrency: current model uses half-duplex approximation; multi-AP spatial concurrency requires ns-3 Level 3 (M APs, N clients)
5. Time-series simulation: Phase 2 of v3 — user defines load profile (T → concurrency %) → simulator runs ticks → outputs time-series chart
6. ns-3 time-series: Simulator::Schedule() for client arrivals/departures, FlowMonitor at each interval — this is Track B, not Sprint 2

## ARCHETYPE NAMES FROM JUNE 3 DELIBERATIONS
(from project_track_b_calibration.md and Google AI reconciliation)
- Stadium (current)
- Convention Center ("a mobile stadium" — uplink-heavy, reconfigurable, Google AI named this)
- Airport
- Hospital
- University
See archetype_matrix_v0.html for full matrix if it exists
