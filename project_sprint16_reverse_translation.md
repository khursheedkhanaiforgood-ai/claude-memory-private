---
name: Sprint 16+ — Translation Direction Roadmap
description: All planned translation directions beyond Cisco→EXOS (Sprint 14). "Cisco outward" first, reverse directions later.
type: project
---

# Translation Direction Roadmap

**Policy (as of 2026-04-03):** "For now only CISCO outward" — Khursheed wants all Cisco-as-source directions first, then reverse.

---

## Topology Parser Coupling Rule
Each translation direction requires TWO parsers:
- **Before parser** — matches source OS syntax → `extract_topology_*()`
- **After parser** — matches target OS syntax → `extract_topology_*()`

Gating logic lives in `config_translator.py` → topology expander section:
```python
# Before — keyed on _SK_SRC_OS
# After  — keyed on _SK_TGT_OS
if _tgt_os_key == "extreme_exos":   → extract_topology_from_exos()
if _tgt_os_key == "extreme_voss":   → extract_topology_from_voss()   # TODO Sprint 17
if _tgt_os_key == "extreme_slxos":  → extract_topology_from_slx()    # TODO Future
# Reverse: if _src_os_key == "extreme_exos": Before → extract_topology_from_exos()
```

---

## Sprint 14 — CISCO → EXOS (DONE)
- Source: Cisco IOS / IOS-XE / NX-OS
- Target: ExtremeXOS 33.x
- Parsers: `parse_config()` + `extract_topology()` (before) / `extract_topology_from_exos()` (after)
- System prompt: `prompts/system_config_translator.md`
- Status: ✅ Live on `feature/sprint14-config-translator`

---

## Sprint 17 — CISCO → VOSS (Next "Cisco outward")
- Source: Cisco IOS / IOS-XE
- Target: VOSS / Fabric Engine 9.x
- Needs:
  1. New system prompt: `prompts/system_config_translator_cisco_to_voss.md`
     - VOSS I-SID for L2 fabric: `vlan i-sid <vlanid> <isid>`
     - IS-IS/SPBM: `router isis` + `auto-sense enable` (no Cisco equivalent)
     - ZTF: `auto-sense enable` → document as VOSS-only
     - VOSS port config: `interface GigabitEthernet <slot/port>` (same modal as Cisco, easier)
  2. `extract_topology_from_voss()` in `cisco_config_parser.py`
     - VOSS is modal like Cisco but with `router isis` fabric blocks
     - Parse `vlan i-sid` for fabric segmentation
  3. Flip `extreme_voss` in `TARGET_OS_OPTIONS` to `active: True`
- Est. effort: 2-3 days (system prompt bulk of work; parser reuses Cisco modal patterns)
- **How to apply:** When user asks about CISCO→VOSS, reference this sprint design.

---

## Sprint 18 — CISCO → SLX-OS (Stretch)
- Source: Cisco IOS-XE / NX-OS
- Target: SLX-OS 20.x
- Prerequisite: SLX-OS feature command parity research
- Flip `extreme_slxos` to `active: True` when ready
- Est. effort: 3-4 days

---

## Sprint 19 — EXOS → CISCO (Reverse — deferred until Cisco outward complete)
**Status:** Deferred — design captured, not started
**Why:** Khursheed said "for now only CISCO outward" (2026-04-03). Architecturally the DB supports bidirectional mappings.

### What's needed
1. New system prompt: `prompts/system_config_translator_exos_to_cisco.md`
   - EXOS flat verb-noun → Cisco modal sub-mode syntax
   - `create vlan DATA tag 10` → `vlan 10 / name DATA`
   - `configure vlan DATA add ports 1 untagged` → `interface Gi1/0/1 / switchport access vlan 10`
   - `configure vlan DATA add ports 1 tagged` → `switchport mode trunk / switchport trunk allowed vlan ...`
   - `configure vlan DATA ipaddress 10.10.10.1/24` → `interface Vlan10 / ip address 10.10.10.1 255.255.255.0`
   - `enable stpd s0 auto-edge ports 1` → `spanning-tree portfast`
   - `enable ssh2` → `line vty 0 15 / transport input ssh`
   - `configure snmp add community readonly STRING` → `snmp-server community STRING RO`
   - EXOS port notation `1:1` → Cisco `GigabitEthernet1/0/1`
2. Add `extreme_exos` to `SOURCE_OS_OPTIONS` with `active: True`
3. Before parser: `extract_topology_from_exos()` (already written)
4. After parser: `parse_config()` + `extract_topology()` (already written)
5. System prompt selection in `config_translate_agent.py`:
   ```python
   if source_os in ("extreme_exos", "extreme_voss"):
       system_prompt = _get_system_prompt("exos_to_cisco")
   else:
       system_prompt = _get_system_prompt("cisco_to_exos")  # existing
   ```
- DB is already bidirectional — RAG lookup for EXOS→Cisco just queries `extreme_exos` column, returns `cisco_iosxe`
- Est. effort: 3-4 days

---

## Sprint 20 — VOSS → CISCO (Reverse — deferred)
- Needs `extract_topology_from_voss()` (Sprint 17 deliverable reused)
- Same system prompt pattern as EXOS→Cisco but with VOSS I-SID/IS-IS specifics
- Est. effort: 2-3 days (system prompt + source detection)

---

## DB Bidirectionality (already ready)
Each row has both `extreme_exos` and `cisco_iosxe` columns filled.
RAG lookup for reverse direction just swaps `source_col` and `target_col` in `_rag_lookup()`.
No DB migration needed for any direction.

**How to apply:** When Khursheed asks about any translation direction other than Cisco→EXOS, refer to this roadmap. Always confirm "Cisco outward first" policy before starting reverse-direction work.
