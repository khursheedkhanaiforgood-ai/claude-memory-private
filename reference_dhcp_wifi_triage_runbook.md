---
name: DHCP / Wi-Fi Client Triage Runbook — AP / XIQ / EXOS
description: Standalone playbook for "Wi-Fi client can't get DHCP" failures. 5-stage triage flow with commands per layer. Distilled from May 6 incident + May 7 Socratic diagnosis. Use BEFORE rebooting — reboot consumes evidence.
type: reference
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## When to use this runbook
Symptom: a Wi-Fi client (or all clients) on an Extreme AP3000 can't get a DHCP lease, or loses one mid-session, or other clients break when a new client joins.

## Golden rule
**Capture diagnostics BEFORE rebooting.** Reboot wipes volatile in-memory state (TK/GTK keys, AP bridge tables, association state) — which IS often the fix, but also IS the evidence. Run the AP-side captures first, *then* reboot if needed.

---

## 5-stage triage flow

### Stage 1 — Confirm the client is actually sending DISCOVERs
Capture from the client itself (the only place you see what the client thinks it's sending).

- macOS: `Wireless Diagnostics → Capture` (Option-click WiFi icon) on `en0`
- iOS: tether through MacBook, capture on tethered interface
- Filter: `bootp || dhcp` in Wireshark

| What you see | What it tells you |
|---|---|
| **No DISCOVERs at all** | Client-side issue. Wi-Fi adapter, OS supplicant, or never associated. STOP — go to Stage 2 to verify association. |
| **DISCOVERs going out, no OFFERs** | Client is associated but bidirectional path is broken somewhere AP↔server. Continue. |
| **DISCOVER → OFFER but no ACK** | One-way works, other broken. Likely intermittent encryption or unicast vs broadcast asymmetry. |

---

### Stage 2 — Did the AP see the client and finish the 4-way handshake?
**XIQ → Device 360 → AP1 → Client List** OR **AP CLI**.

```
show client                          # all associated clients with state
show client mac <client-mac>         # per-client deep dive
```

| Client state column | Diagnosis |
|---|---|
| Not listed | Client never associated. Check SSID broadcast, RF coverage, supplicant config. |
| `Associated` (no auth) | Open network or pre-EAPOL. Not normal for WPA2/3 SSID. |
| `4WAY-PENDING` | **Smoking gun for option (b) WPA2 key state.** EAPOL handshake started but didn't complete. TK never installed. |
| `Authenticated / Data-OK` | Client looks fully up. Move to Stage 3. |

`show client mac <MAC>` should also show `TK Installed: yes/no`. If `no` while state is `Data-OK` → contradiction = client-AP key desync, possibly stale.

---

### Stage 3 — Are the client's frames making it past the AP onto the wire?
**EXOS (SW1) — does SW1's FDB on the client's VLAN have the client MAC?**

```
show fdb vlan VLAN_20_Corporate_Wireless         # learned MACs on this VLAN
show fdb port 3                                  # learned MACs on the AP-facing port
show iparp vlan VLAN_20_Corporate_Wireless       # ARP table for this VLAN
```

| What you see | Diagnosis |
|---|---|
| Client MAC in FDB on Port 3 | Frames are reaching SW1. Skip to Stage 4. |
| Client MAC NOT in FDB on any port | Frames are dying between AP RX-radio and AP TX-wired. **Check decrypt counters at AP.** This is what happened on May 6 — bad TK install, AP couldn't decrypt MacBook frames, dropped them silently. |
| Client MAC on Port 3 but NOT in ARP | L2 reaches SW1 but L3 broken. Check VLAN ipforwarding flag, gateway SVI status. |

**AP-side decrypt counter check (the smoking gun for silent-drop scenarios):**
```
show counters wireless                   # or `show wlan-statistics` per platform
show interface wifi0 statistics          # per-radio RX OK / errors / decrypt failures
```

Procedure: capture, wait 30 s, capture again, diff:
```bash
ap-cli show counters wireless > t0.txt
sleep 30
ap-cli show counters wireless > t30.txt
diff t0.txt t30.txt | grep -i "decrypt\|mic"
```

A spiking decrypt-error / MIC-failure counter for the affected radio = **WPA2 key-state wedge confirmed in real-time** (option b). Counter unchanged = look elsewhere.

---

### Stage 4 — Did the DHCP server see the request and have an address to give?
**EXOS (SW1) — DHCP server state.**

```
show dhcp-server                         # all VLANs, pools, leases, port-bindings
show dhcp-server VLAN_20_Corporate_Wireless leases
show vlan VLAN_20_Corporate_Wireless     # ipforwarding flag, port membership
show log                                 # filter for DHCP / DHCPDISCOVER
```

Things to verify:

- **Pool not exhausted** — count assigned vs range. If full, expand or reduce `dhcp-lease-timer`.
- **DHCP enabled on AP-facing port** — `Ports DHCP Enabled: 3,10` line. If port 3 missing, requests are silently ignored even though daemon is up.
- **`enable ipforwarding vlan X`** — without this, clients get IPs but no internet. Karl Rule.
- **No stale lease for this MAC** — old lease at IP X.X.X.100 with this MAC? DHCP may try to unicast OFFER to X.X.X.100 which the client doesn't own yet.

If pool good, port enabled, but no DHCP log entries for this MAC's DISCOVER → the request never arrived at the daemon. Recheck Stage 3.

---

### Stage 5 — Are OFFERs reaching the client (downstream encryption)?
This is the half most people miss because they only capture upstream from the client. To check downstream you need an **AP packet capture** (XIQ → Device 360 → Tools → Capture).

Filter: `wlan.addr == <client-mac>`

| What you see | Diagnosis |
|---|---|
| OFFER sent by AP, `Protected Frame=1` | AP is forwarding it. Client should see it. If client capture shows no OFFER, check **GTK desync** (broadcast OFFERs encrypted with GTK_v2, client still on GTK_v1). |
| No OFFER ever sent | Frame never reached AP downstream queue. Stage 4 issue, not Stage 5. |
| OFFER sent but as `Protected Frame=0` (cleartext) | Encryption mismatch — usually a configuration error, not a key wedge. |

**GTK desync test:** force the client to disconnect & re-associate (toggle Wi-Fi off/on). If DHCP succeeds on re-assoc but not before → GTK was the wedge (it's per-BSSID, gets refreshed on every new association from your client).

---

## Quick-reference: the four "smoking gun" indicators

| Indicator | Where to find | Means |
|---|---|---|
| Client state stuck at `4WAY-PENDING` | XIQ → Device 360 → Client list, or AP `show client` | TK never installed → option (b) |
| **Decrypt-error counter spiking on AP radio** | AP `show counters wireless` (diff over 30 s) | **THE confirmation for (b) — encrypted frames received but undecodable** |
| Client MAC absent from SW1 `show fdb port 3` | EXOS | Frames dying upstream of SW1 (at AP) — corroborates (b) |
| OFFERs visible in AP capture but invisible in client capture | XIQ AP capture vs client en0 capture | GTK desync — broadcast OFFER encrypted with mismatched key |

---

## XIQ web-UI shortcuts (when CLI feels heavy)
- **Device 360 → Client tab** — see clients without SSH'ing in
- **Live Log Stream** (per-device) — filter by client MAC, see association events in real-time
- **Network Policy → SSID → Security** — verify encryption matches what client expects (WPA2 vs WPA3, PSK length, etc.)
- **Tools → Capture** — start a wireless capture on the AP itself, download .pcap

---

## What NOT to do first
- Don't reboot anything until you've grabbed at least the AP `show client` + `show counters wireless` snapshot. The reboot is often the fix, but it's also the evidence-eraser.
- Don't blame DHCP server config until you've confirmed frames are reaching SW1's FDB. Most "DHCP problems" turn out to be Wi-Fi-encryption problems.
- Don't trust client-side error messages — macOS/iOS will say "Connection failed" for a dozen different root causes.

---

## Cross-references
- Memory: `project_dhcp_macbook_incident.md` — the May 6 incident this runbook is distilled from
- Memory: `project_eapol_forensics_may4.md` — TK/GTK theory deep dive
- EOD HTML: `session_summary_20260507.html` Panel 8 (5-step XIQ procedure) and Panel 7 (validation pcap)
- Repo data: `docs/data/may7-dhcp/tech_support_AP1_May7_2026.txt` — example AP3000 show-tech (debug + radio + wipsd logs)
- Repo data: `docs/data/may7-dhcp/tech_support_SW1_EXOS_May7_2026.txt` — example EXOS show tech-support