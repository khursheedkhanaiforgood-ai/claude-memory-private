---
name: reference_iqcontroller_vs_iqengine
description: KB-confirmed terminology split between IQEngine (AP OS/firmware persona) and IQController/XIQ-C (on-prem controller platform) — resolves a recurring confusion point
type: reference
originSessionId: b60a6aab-a768-4fff-b217-9bd45e71c887
---
**IQEngine** = the Access Point operating system / cloud-native AP software persona (formerly HiveOS). It runs *on* the AP. Not a management platform.

**IQController / ExtremeCloud IQ Controller / XIQ-C** = a separate, centralized **controller** product for on-prem campus deployments — airgap licensing, tunneling traffic, distributed or centralized data planes, large mesh support, integrates with XIQ-Site Engine / ExtremeControl / ExtremeAnalytics / AirDefense. Distinct product line from cloud-managed EP1/XIQ.

These are not interchangeable names for the same thing: one is AP firmware, the other is an on-prem controller. Confusion between them surfaced 2026-09-02 while fact-checking a pasted 3rd-party AI transcript about PPSK/I-SID mapping for medical-IoT segmentation (see `project_voss_fabric_migration.md` → "IQController vs IQEngine" section, same date).

Source: `Solutions Design2026StudentGuide26.2.1.pdf` (Sales AI Assistant) + extremenetworks.com product pages (ExtremeCloud IQ Controller, ExtremeCloud IQ).
