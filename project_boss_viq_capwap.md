---
name: Boss Karl's VIQ — New AP CAPWAP + PPSK Deployment
description: New task assigned May 13: provision a new AP in boss Karl's VIQ instance and deploy a CAPWAP-managed PPSK policy from scratch.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---

## Task
Provision a new AP in boss Karl's lab VIQ (separate XIQ org from MyLab_XIQ).
Deploy CAPWAP connection + PPSK SSID policy. This is a full AP onboarding from scratch.

## Why
Boss has granted access to his VIQ. This is a client-readiness exercise — demonstrates
ability to onboard AP and deploy PPSK in a production-style XIQ org.

## CAPWAP AP Onboarding Sequence
1. AP gets IP via DHCP
2. AP discovers XIQ: DHCP Option 43 → DNS (redirect.aerohive.com) → manual entry
3. AP establishes CAPWAP tunnel (DTLS handshake, UDP 5246/5247)
4. AP appears in XIQ dashboard (Monitor → Devices) — status: "Connected"
5. Assign AP to Network Policy (with PPSK SSID configured)
6. XIQ pushes policy to AP via CAPWAP
7. PPSK SSID broadcasts, clients can connect

## Key RFCs
- RFC 5415 — CAPWAP core spec
- RFC 5416 — CAPWAP 802.11 binding
- RFC 5417 — DHCP Option 43 for AP discovery
- RFC 6347 — DTLS (CAPWAP tunnel security)

## Status — COMPLETE (May 14 2026)
- Access to boss's VIQ: granted ✅
- AP onboarded: AP3000, serial HA012519Y-10623 ✅
- Firmware updated: white LED during update, rebooted clean ✅
- Policy deployed: KB_School_AP1_VLAN10 (same as MyLab policy) ✅
- SSID: KB_School_Broadcast, PPSK, Staff_PPSK + Students_PPSK, VLAN10 ✅
- iPhone E2E: 10.10.0.10, internet confirmed ✅

## Onboarding Sequence (confirmed working)
1. Factory reset AP from MyLab XIQ (Reset Device to Default)
2. Delete AP from MyLab XIQ device list (free the serial)
3. Add serial HA012519Y-10623 in Karl's VIQ (Admin → Add Devices)
4. AP boots → white LED (firmware update) → reboot → orange → green
5. Create/assign policy in Karl's VIQ → deploy → SSID broadcasts
6. DHCP from SW1 VLAN10 pool (10.10.0.x), internet via QF-Modem

## Key Lesson
White LED = firmware update in progress. Do NOT unplug. Wait for green.
AP must be DELETED from old XIQ org before onboarding in new org.
