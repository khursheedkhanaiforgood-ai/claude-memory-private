---
name: WiFi Digital Twin — Branch Strategy & Generation Decision
description: Two-branch strategy for WiFi 6 vs WiFi 7. Mixed-mode (co-existing generations) deferred to Sprint 4+ — requires floor plan topology. Strategy pattern architecture confirmed.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Decision: Uniform Generation per Site (Phase 1)

User picks AP generation at DPM time. Tool designs uniformly for that generation across the site. No mixed AP models in Phase 1.

**Why:** Mixed WiFi 6 + WiFi 7 on same site creates zone-boundary problems (MLO drops on roam, 320→160 MHz ACI at adjacency, Multi-AP Coord cluster = WiFi 7 only). Resolving these requires knowing which AP is adjacent to which — needs floor plan + adjacency graph, which Phase 1 does not collect.

**How to apply:** Never conflate WiFi 6/7 mixed-mode with the two-branch strategy. Mixed-mode is a separate future sprint.

---

## Two Branches

| Branch | AP Fleet | MCS ceiling | Key WiFi 7 features |
|--------|----------|-------------|-------------------|
| `feature/wifi6-baseline` | AP3000 / AP4000 | MCS 11 (1024-QAM) | None — clean WiFi 6 |
| `feature/wifi7-complete` | AP5020 / AP5022 / AP5060 | MCS 13 (4K-QAM, SNR≥36dB) | MLO, 320MHz, Multi-AP Coord, Puncturing, HARQ |

Comparison Engine: runs both strategies on same DPM → outputs delta report.

---

## Strategy Pattern (code contract)

```python
class WiFi6LinkBudget(LinkBudgetStrategy):
    def max_mcs_index(self) -> int: return 11

class WiFi7LinkBudget(WiFi6LinkBudget):   # inherits WiFi 6
    def snr_to_mcs(...):
        result = super().snr_to_mcs(...)   # WiFi 6 first
        if snr_db >= 36 and self.be_capable:
            result = self._apply_mcs_12_13(snr_db)
        return result
    def max_mcs_index(self) -> int: return 13

class ComparisonEngine:
    def run(self, dpm) -> ComparisonReport:
        wifi6 = DesignEngine(strategy=WiFi6Strategy()).run(dpm)
        wifi7 = DesignEngine(strategy=WiFi7Strategy()).run(dpm)
        return ComparisonReport(wifi6=wifi6, wifi7=wifi7, delta=self._diff(wifi6, wifi7))
```

---

## Mixed-Mode (future feature/mixed-topology)

Deferred to Sprint 4+. Will need:
- Floor plan upload (SVG/DXF/image)
- AP placement drag-and-drop tool
- Adjacency graph derived from placement
- Per-zone AP model assignment
- Zone-boundary channel width reconciliation (320→160 MHz)
- `MixedAPStrategy = zone_map[zone] → strategy`
- Comparison Engine mode 3 (mixed vs pure 6 vs pure 7)

**Why deferred:** Without adjacency graph, cannot predict MLO loss on inter-generation roam or ACI at zone boundaries. The boundary problems make the analysis incorrect, not just incomplete.
