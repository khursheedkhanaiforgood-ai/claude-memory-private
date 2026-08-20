---
name: VOSS Site Engine Lab — voss_policy + I-SID for AP Traffic
description: Future lab blocked on IS-IS adjacency restoration in EVE-NG SD-WAN. Site Engine push of voss_policy to enable I-SIDs for AP traffic on 5320_fabric_se.
type: project
originSessionId: aug-20-2026
---

## Lab Goal
Configure XIQ Site Engine (XIQ_SE) to push a voss_policy to 5320_fabric_se that enables specific I-SIDs to pick up AP traffic.

## Dependency Chain
```
EVE-NG disk fix (Karl) 
  → IS-IS adjacency restored (vBOB02-Headend ↔ vBOB02-Kit)
  → SD-WAN overlay up (purple map line active)
  → Site Engine voss_policy push
  → I-SIDs active for AP traffic on 5320_fabric_se
  → AP connects via FA → VLANs provisioned → clients on service
```

## Current Blocker
- vBOB02-Headend: Appliance Disk Failure (Aug 19 19:00) → ISIS Adjacency Lost (Aug 19 19:20)
- vBOB02-Kit: Appliance Disk Failure (Aug 19 19:37)
- Karl fixing EVE-NG infrastructure
- IS-IS adjacency must restore before any policy push

## Active Alarms (as of Aug 20 2026)
| Severity | Alarm | Appliance |
|---|---|---|
| Critical | ISIS Adjacency Lost | vBOB02-Headend |
| Major | Appliance Disk Failure | vBOB02-Headend |
| Major | Appliance Disk Failure | vBOB02-Kit |

## Switches Involved
- 5320_fabric_se — VOSS, managed by XIQ Site Engine (on-prem)
- 5320_Exos_EP1 — EXOS → to be converted to VOSS, managed by EP1

## Management Split
- XIQ_SE (Site Engine) → manages 5320_fabric_se + SD-WAN/EVE-NG
- XIQ_C (EP1 cloud) → manages 5320_Exos_EP1 + AP_3000

## How to Apply
When resuming: verify IS-IS adjacency restored first (`show isis neighbors` on headend), then proceed to Site Engine voss_policy configuration.
