---
name: Simulator v2 Agentic Framework Design
description: LangGraph multi-agent architecture for stadium WiFi simulator — orchestrator + 7 sub-agents, sprint workflow automation
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---

## Intent (June 3 2026)
Build a Python agentic framework (LangGraph + Claude SDK) that orchestrates simulator development and operation. User wants an orchestrator agent coordinating specialized sub-agents instead of manually running sprint tasks.

## ⚠️ SEQUENCING — CRITICAL
**DO NOT build the agentic framework until physics is fixed.**
Order: Sprint 1 (fix physics) → Sprint 2 (validate QoE curves) → THEN scaffold agentic framework → Monte Carlo → Optimizer → ns-3 calibration.
User's exact words: "I want to fix the physics first. When I thought about an orchestrator, it would be for after we fixed the physics bugs. Then we could get into a flow."

**Why:** Simulator has 4 sprint layers (physics fix → param validation → Monte Carlo → optimizer → ns-3 calibration). Each layer has distinct tools and concerns. An orchestrator can run them autonomously, validate outputs, and chain sprints without manual intervention.

## Stack
- **LangGraph** for orchestration state machine (user already has this in deep-research-engine)
- **Claude SDK** per sub-agent (Sonnet for most, Opus for physics audit + calibration)
- **Python / NumPy** for Monte Carlo compute (100× faster than JS)
- **HTML simulator** stays as UI layer — agents write back to it

## Shared State Object
```python
SimulatorState = {
    "params": {},           # current 60-param config
    "scenario": {},         # current scenario conditions (fan count, concurrency, etc.)
    "physics_audit": [],    # list of found/fixed bugs with line numbers
    "mc_results": {"P5": {}, "P50": {}, "P95": {}},
    "optimizer_result": {}, # best param set + score
    "calibration_error": float,  # Track A vs ns-3 delta
    "validation_level": int,     # 1–4 (PHY / MAC / spatial / mobility)
    "pending_tasks": [],
    "completed_tasks": []
}
```

## Agents

### OrchestratorAgent (Lead)
- Entry point: receives user intent ("fix physics", "optimize kickoff", "run full calibration")
- Maintains SimulatorState, routes to sub-agents, aggregates outputs
- Decides next agent based on state (e.g., if physics_audit has unfixed bugs → PhysicsAgent first)

### PhysicsAgent
- Input: simulator_v2.html compute() function
- Function: audit + fix physics bugs (C1/C2/C3 and future bugs)
- Output: corrected JS + validation report (airtime 20-60%? capGbps 1400-3500?)
- Tools: read_file, edit_file, validate_physics_outputs

### ParameterAgent
- Input: spec docs, current 60 params, applyOptimal() values
- Function: validate params against IEEE 802.11be spec, flag conflicts
- Output: validation table, corrected applyOptimal() values (4 known conflicts)
- Tools: read_spec, compare_param, flag_conflict

### MonteCarloAgent
- Input: parameter set θ, scenario conditions
- Function: N-iteration Poisson+Pareto simulation → QoE distributions
- Output: P5/P50/P95 for each metric (score, POS latency, video rebuffer, VoIP MOS)
- Tools: run_simulation, compute_percentiles, generate_qoe_curves
- **Note:** runs in Python NumPy, not JS — 100× faster for large N

### OptimizerAgent
- Input: objective function, search space (15 knobs after promotion)
- Function: Bayesian or genetic search over non-trivial parameter space
- Output: ranked golden sets with P5/P50/P95 QoE bands
- Tools: run_monte_carlo, evaluate_objective, rank_configs
- Objective: maximize P{n}[U(θ)] - 0.5*(P95-P5) s.t. P(posLat>threshold) ≤ 1%

### ScenarioAgent
- Input: event timeline (kickoff, halftime, wave, final whistle)
- Function: generate 4-level validation scenario matrices
- Output: structured test suite (1AP→multi-AP→multi-client→mobility)
- Tools: generate_scenarios, parameterize_density_sweep, run_validation_sequence

### CalibrationAgent (future — needs ns-3)
- Input: ns-3 run outputs + Track A predictions for same θ
- Function: compute calibration error δ per scenario, adjust capMult coefficients
- Output: calibrated coefficient table, updated simulator_v2.html multipliers
- Tools: compare_outputs, adjust_coefficients, write_calibration_report

### UIAgent
- Input: physics/MC/optimizer outputs
- Function: update HTML simulator display with new results, fix visual bugs
- Output: updated stadium_wifi_simulator_v2.html
- Tools: read_html, edit_html, inject_results, validate_display

### ReportAgent
- Input: all agent outputs, SimulatorState
- Function: generate EOD summary HTML + golden set parameter report
- Output: session_summary_YYYYMMDD.html, commit + push
- Tools: aggregate_findings, write_html, git_commit, git_push

## Sprint 1 Workflow (automated)
```
User: "run Sprint 1"
→ Orchestrator → PhysicsAgent: fix C1+C2+C3+L1
→ PhysicsAgent → validate: airtime 20-60%? capGbps 1400-3500?
→ Orchestrator → ParameterAgent: flag applyOptimal() conflicts
→ Orchestrator → MonteCarloAgent: baseline QoE curves with fixed physics
→ Orchestrator → UIAgent: apply all changes to simulator_v2.html
→ Orchestrator → ReportAgent: Sprint 1 EOD summary
```

## Repo Location
New Python project: `/Users/khukhan/5320-onboarding-agent/sim_agents/` (co-located with simulator)
OR: separate repo `simulator-agentic-framework`
Decision pending — discuss with user at Sprint 1 start.

## How to apply
Before building Track B (ns-3 calibration), frame each ns-3 run as a CalibrationAgent task. The agent receives (θ, scenario) → runs ns-3 → returns QoE distribution → compares to Track A → outputs δ. This makes the calibration loop repeatable and auditable.
