---
name: QoS Egress Validation — May 8 2026 (separate branch from DHCP)
description: Boss claim "all traffic best-effort despite XIQ priority" validated and decomposed into 3 findings. Apr 21 lab work mapped VLAN→QP but never set queue weights. EXOS VLAN qosprofile is ingress-only — downstream L3-routed traffic stays in QP1. YouTube CDN doesn't mark DSCP. Branch: feature/qos-egress-validation.
type: project
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Branch
`feature/qos-egress-validation` — separate from DHCP work on `feature/auto-deploy-agent`. All QoS-related artifacts in `docs/data/may8-qos/`.

## Date
2026-05-08 PM

## Lab setup recap (from project_karl_lab_apr21)
- SW1 (5320-16P-2MXT-2X-SwitchEngine), Port 3 = AP1, Port 1 = QF Modem, Port 5 = wired client, Port 10 = mirror dest (during testing)
- AP1 = AP3000, serves Corporate (VLAN 20, 5GHz ch161) + Guest (VLAN 30) + OWE-Guest
- VLAN-to-qosprofile mapping configured Apr 21:
  - VLAN 10 (Wired Corp) → QP6
  - VLAN 20 (Wi-Fi Corp) → QP6
  - VLAN 30 (Wi-Fi Guest) → QP1
- XIQ User Profile scheduling weights: Corp=100, Guest=10 (these are AP-side WMM only — no XIQ→EXOS translation)

## THE THREE FINDINGS

### Finding #1 — FIXED in lab — SW1 queue weights all = 1 system-wide
```
show qosprofile (initial state)
QP1   Weight = 1
QP6   Weight = 1
QP8   Weight = 1
```
The Apr 21 lab work mapped VLANs to queues but **never differentiated queue weights from default Weight=1**. Queue scheduler treats all queues equally even when traffic IS in different queues.

**Fix applied:**
```
configure qosprofile QP6 maxbuffer 100 weight 10
save configuration
```
After fix: QP1=1, QP6=10, QP8=1. Confirmed via show qosprofile and show port 3 information detail.

### Finding #2 — ARCHITECTURAL — VLAN qosprofile is ingress-side only
**EXOS `configure vlan <name> qosprofile <profile>` classifies based on INGRESS VLAN tag. It does NOT apply at egress when traffic is L3-routed INTO the VLAN from another port.**

Direction asymmetry observed in qosmonitor:

| Direction | Path | Result |
|-----------|------|--------|
| Upstream (client→Internet) | Port 3 ingress (VLAN 20 tag) → QP6 → egress Port 1 in QP6 | ✅ Works (Port 1 QP6 count = 7,807 packets) |
| Downstream (Internet→client) | Port 1 ingress (untagged from QF Modem PCP=0) → QP1 → routed to VLAN 20 → Port 3 egress STAYS in QP1 | ❌ Fails (Port 3 QP6 count = 0 across all tests) |

**For YouTube/streaming workloads** (dominated by downstream traffic), this means VLAN qosprofile mapping has effectively zero effect on the bulk of user-facing traffic.

**Even after disabling 802.1p ingress examination on Port 3** (`disable dot1p examination ports 3`), Port 3 QP6 remained 0. Confirms it's not the 802.1p override — it's the architectural ingress-only nature of VLAN qosprofile.

### Finding #3 — DATA-PLANE — DSCP marking absent end-to-end
Pcap analysis of YouTube traffic (Corp + Guest):
- Corp pcap (work MBP en0, 193K frames, 20 min): **99.85% DSCP=0**
- Guest pcap (personal MBP en0, 135K frames, 8 min): **99.96% DSCP=0**

YouTube CDN doesn't mark video flows. QF Modem doesn't preserve/add DSCP. AP3000 doesn't add DSCP (XIQ User Profile QoS Marker not configured). So DSCP-based classification at SW1 Port 1 ingress (`enable diffserv examination`) won't help — there's nothing to differentiate on.

## qosmonitor evidence (3 snapshots)

Saved verbatim in `docs/data/may8-qos/sw1_qosmonitor_evidence.txt`.

T0 (15:16, before fixes): Port 3 QP1=139,543 QP6=0
T1 (15:21, after weight fix): Port 3 QP1=263,195 QP6=0
T2 (15:27, after disable dot1p): Port 3 QP1=47,395 QP6=0

Port 1 (upstream) QP6 incrementing: 0 → 1,746 → 7,807. Upstream works.
Port 3 (downstream) QP6 stuck at 0. Downstream broken.

## Pcap files (trimmed to fit GitHub size limit)

| File | Size | Contains |
|------|------|----------|
| `docs/data/may8-qos/corp_workmbp_en0_first50k.pcapng` | ~50 MB | Corp YouTube on work MBP, first 50K frames |
| `docs/data/may8-qos/guest_personalmbp_en0_first50k.pcapng` | ~46 MB | Guest YouTube on personal MBP, first 50K frames |
| `docs/data/may8-qos/sw1_qosmonitor_evidence.txt` | ~6 KB | All SW1 evidence + config diffs |
| `docs/data/may8-qos/dialogue_qos_validation_May8_2026.md` | ~12 KB | Verbatim Q&A for GTAC escalation |

Original full-size sources (kept locally, NOT committed):
- `/Users/khukhan/Library/CloudStorage/OneDrive-ExtremeNetworks,Inc/Desktop/ap1_May8_qos_1405PST.pcapng` (181 MB Corp)
- `/Users/khukhan/Downloads/QoS_May8.pcapng` (135 MB Guest)

## Live URLs (after publish)
- EOD: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_20260508_qos.html
- Verbatim dialogue: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may8-qos/dialogue_qos_validation_May8_2026.md
- Evidence: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may8-qos/sw1_qosmonitor_evidence.txt

## XIQ UX gap finding (worth surfacing to R&D)

User Profile "Scheduling Weight" field affects only AP-side WMM. XIQ does NOT auto-translate this to EXOS switch queue weights. Three additional configurations are needed but UI doesn't make this clear:

1. User Profile → QoS Marker (Class of Service / DSCP)
2. Switch Template → QoS Profiles (queue weights)
3. WAN-side ACL classification (for downstream)

Customer reasonably expects "Scheduling Weight 100 vs 10" to enforce end-to-end priority. It only enforces wireless airtime. Other layers must be configured deliberately.

## Pending work — Option C path B (next)
Apply ACL-based classification on Port 1 ingress to match Corp dest subnet (10.20.0.0/24) and assign QP6. Then re-test under load.

```
# Conceptual EXOS access-list policy
edit policy "Corp_Priority"
entry Corp_DL_to_QP6 {
  if match all { destination-address 10.20.0.0/24 ; }
  then { qosprofile QP6 ; }
}
configure access-list "Corp_Priority" ports 1 ingress
save configuration
```

## Cross-references
- `project_karl_lab_apr21.md` — original Apr 21 QoS configuration
- `project_dhcp_macbook_incident.md` — separate incident on same lab
- `reference_5320_pages_source.md` — main branch publishes Pages

## Status
- [x] Three findings documented
- [x] Fix #1 (weight differentiation) applied + saved on SW1
- [x] Fix #2 (disable dot1p exam Port 3) applied — confirmed not sufficient
- [x] EOD HTML + verbatim dialogue + evidence saved to repo
- [ ] Push to feature/qos-egress-validation
- [ ] Cherry-pick to main for publishing
- [ ] Update landing page links
- [ ] Apply Option C path B (ACL classification) + re-test under load
