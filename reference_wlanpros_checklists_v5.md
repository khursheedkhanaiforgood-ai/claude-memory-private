---
name: WLANPros Wi-Fi Checklists v5 — All 8 Sheets
description: Complete content of WLANPros Wi-Fi Checklists v5 (1).xlsx — 8 tabs covering Top 20, Extended, Connection, Not-Wireless, Passpoint, WiFi 7, eduroam, Apple Best Practices. © 2025 WLAN Pros.
type: reference
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## File Location
`/Users/khukhan/Downloads/WLPC Troubleshooting/WLANPros Wi-Fi Checklists v5 (1).xlsx`
**PRIVATE — never commit to GitHub**

## Checklist Usage
- Enter 1 if item meets requirements, 0 if not
- Green boxes = variable metrics (adjust to site requirements)
- ✅ = should be ON / positive | ❌ = not required / turn OFF | 🛜 = variable metric

---

## SHEET 1: Wi-Fi Top 20 v3 (Items 1–20, per band)

| # | Item | 2.4GHz | 5GHz | 6GHz | Notes |
|---|------|--------|------|------|-------|
| 1 | Captive Web Portal | 🛜 Only if required | 🛜 Only if required | 🛜 Only if required | CPs add complexity + friction; don't start until AFTER network working |
| 2 | 1, 6, 11 Only | ✅ Ch 1,6,11 ONLY | ❌ N/A | ❌ N/A | Never use other 2.4GHz channels |
| 3 | 20 MHz Channels | ✅ 20MHz ONLY | 🛜 Use | 🛜 Use | Never 40MHz on 2.4GHz |
| 4 | 40 MHz Channels | ❌ No 40MHz | 🛜 Check CCC | 🛜 Use | Use widest until CCI appears |
| 5 | 80 MHz Channels | ❌ N/A | ❌ Not Recommended | 🛜 Check CCC | Use widest until CCI |
| 6 | CCI threshold | 🛜 >-85dBm | 🛜 >-85dBm | 🛜 >-85dBm | Minimize CCI — wastes airtime |
| 7 | OBSS Consistent Channel Bonding | ❌ No bonding | ✅ High side consistent | ✅ High side consistent | 6GHz primary = PSC channel |
| 8 | DFS in Use | ❌ N/A | 🛜 Use & Review | ❌ N/A | Use DFS until you can't |
| 9 | Min Basic Rates (MBR) | 🛜 ≥12Mbps, No b | 🛜 ≥12Mbps | 🛜 ≥12Mbps | Start 12, try 24 in high density; remove 1,2,5.5,11 on 2.4GHz |
| 10 | SSIDs Across Bands | ❌ No 2.4-only SSIDs | ✅ Use w/RNR | ✅ Use w/RNR | 6GHz: use RNR for out-of-band discovery |
| 11 | Primary RSSI | 🛜 -67dBm | 🛜 -65dBm | 🛜 -65dBm | Set per client device requirements |
| 12 | Secondary RSSI | 🛜 -67dBm | 🛜 -65dBm | 🛜 -65dBm | For full AP redundancy match primary |
| 13 | AP TX Power | 🛜 7dBm target | 🛜 13dBm target | 🛜 16dBm target | Plan strategy for min TX power |
| 14 | WPA2/3 Transition | ❌ Not recommended | ❌ Not recommended | ❌ NOT ALLOWED | Get off transition ASAP |
| 15 | WPA3 | 🛜 Recommended | 🛜 Recommended | 🛜 REQUIRED | WPA3 mandatory in 6GHz |
| 16 | 802.11k Neighbor Report | ✅ | ✅ | ✅ | APs scan + share neighbor results |
| 17 | 802.11r Fast Transition | ✅ | ✅ | ✅ | Faster roaming; may cause issues with legacy |
| 18 | 802.11v Transition Management | ✅ | ✅ | ✅ | Roaming, load balancing, band steering |
| 19 | Channel Utilization/BSS Load | 🛜 <40% | 🛜 <20% | 🛜 <20% | Critical RF health metric |
| 20 | Clients per Radio (BSS Load) | 🛜 <40 | 🛜 <40 | 🛜 <40 | Standard density; LPV may be higher |

---

## SHEET 2: Wi-Fi Extended v4 (Items 21–79)

Key items (abbreviated):
| # | Item | Recommendation | Notes |
|---|------|---------------|-------|
| 21 | Beacon Interval | ✅ 102.4ms | Set and forget — standard default |
| 22 | DTIM Interval | ✅ Default=1 | High DTIM = battery saving; check manufacturer |
| 23 | TKIP/WEP/WPA1 | ❌ Never | Do not use |
| 24 | WPA2 | 🛜 Use on 2.4/5 | ❌ NOT ALLOWED on 6GHz |
| 25 | 802.11b Rates (1,2,5.5,11) | ✅ OFF | Removes 802.11b clients; exceptions: airports, medical, industrial |
| 26 | 40MHz Intolerant | ✅ ON in 2.4GHz | Prevents neighbors from using 40MHz |
| 27 | 160MHz Channels | ❌ 5GHz not recommended | 🛜 Check CCC on 6GHz |
| 28 | 320MHz Channels | ❌ N/A 5GHz | ❌ Not recommended 6GHz | Use widest until CCI |
| 29 | Adjacent Channel Interference | 🛜 >-85dBm | Worse than CCI |
| 32 | 6GHz In-Band Discovery | 🛜 Choose method | FILS, Unsolicited Probe Response, or PSC |
| 33 | 6GHz Low Power Indoor | 🛜 Use | Check regulatory domain |
| 34 | 6GHz Out-of-Band Discovery | 🛜 Choose method | RNR on 2.4/5GHz SSIDs |
| 35 | 6GHz Standard Power | 🛜 Check AFC | Check regulatory domain |
| 36 | 6GHz PSC | 🛜 Use PSC | Helps 6GHz clients find APs |
| 37 | Hidden SSIDs | ❌ Never | No security benefit, causes problems |
| 38 | SSID Count | ✅ ≤4 | One per auth method |
| 39 | WiFi 7 MLO Target SSID | ✅ All bands part of ML group | Even if only 2 of 3 bands used |
| 40 | MLD MAC Advertised | ✅ | Each SSID advertises MLD MAC + Link ID |
| 42 | 802.11d Country Code | ✅ ON | Must match geography |
| 43 | 802.11e QoS (WMM) | ✅ ON | Required for BSS Load IE; without WMM clients limited to 54Mbps |
| 44 | 802.11h TPC | ✅ ON | Helps clients extend battery |
| 45 | 802.11i Security | ✅ No TKIP/WEP | 6GHz = WPA3/OWE ONLY |
| 46 | 802.11w MFP | 🛜 6GHz Support ON | ✅ REQUIRED on 6GHz; mandatory in WPA3 |
| 47 | 802.11u Passpoint/OpenRoaming | ✅ If enabled | Verify RCOIs advertised correctly |
| 48 | MU-MIMO | ❌ Not recommended | OFF unless proven compatible with client devices |
| 49 | TX Beamforming | ❌ Not recommended | OFF unless proven — see Devin Akin's course |
| 50 | OFDMA | 🛜 ON then validate | ON unless causing client issues |
| 51 | Channel Auto Setting | ❌ Not recommended | Study vendor-specific AutoRF before applying |
| 52 | Dynamic Channel Width | ❌ Not recommended | May confuse client roaming algorithms |
| 53 | Client Isolation | ✅ ON in public | Prevents client-to-client traffic |
| 54 | Airtime Fairness | ❌ Not recommended | Minor benefit, potential issues |
| 55 | DFS Recurring Events | 🛜 Check logs | Remove DFS channels if recurring triggers |
| 56 | Mesh | 🛜 <+1 hop | Mesh has airtime cost |
| 57 | AP Background Scanning | ✅ ON | Helps APs find/share neighbor info |
| 58 | Band Steering | ❌ Not recommended | Unnecessary if SSIDs separated per band |
| 59 | Probe Suppression | 🛜 Large venues only | May cause issues in normal density |
| 60 | Auto RF/TX Power | 🛜 Static or controlled | Religious debate — choose static or dynamic based on complexity |
| 61 | Load Balancing | ❌ Not recommended | Issues in normal density |
| 62 | PoE Requirements | 2.4GHz: 802.3af (15.4W) | 5GHz: 802.3at (25.5W) | 6GHz/WiFi7: 802.3bt (51W) |
| 63 | PoE Validate | ✅ Confirm actual draw | Verify PHY rate + PoE class |
| 64 | Non-802.11 Interference | 🛜 >-96dBm | Find and eliminate; 2.4GHz hardest |
| 65 | Firmware Consistent | ✅ All APs same version | |
| 66 | Firmware Updated | ✅ Current firmware | |
| 72-79 | UNII bands (1,2a,2c,3,5,6,7,8) | 🛜 Check regulatory domain | Per geography |

---

## SHEET 3: Wi-Fi Connection v3 (14-Step Post-Deploy Connection Test)
1. Can see all SSIDs being broadcast
2. Target SSID is WiFi 7 w/ MLO
3. Multi-Link Device MAC advertised
4. Associate to target SSID
5. Complete SSID authentication
6. Receive IP address via DHCP
7. Receive Default Gateway + DNS
8. Ping Default Gateway
9. Ping DNS
10. Ping Remote IP address
11. Ping Remote DNS address
12. Check client MCS
13. Check client TX data rate
14. Complete network speed test (non-production only — generates high channel utilization)

---

## SHEET 4: Not Wireless v3 (Pre/Post AP Install — Wired Validation)

**Before Installing AP (use LinkSprinter/LinkRunner AT/EtherScope/laptop):**
1. Cable ≥ Cat5e specs
2. Total cable <100m (including patch cords)
3. Connection link speed confirmed
4. Advertised link speed confirmed
5. PoE standard: 802.3af/at/bt
6. PoE meets AP specific requirements
7. DHCP address + VLAN confirmed
8. Correct VLAN assignment confirmed
9. Access or Trunk port as required
10. Default Gateway confirmed
11. Ping default gateway
12. Target IP addresses reachable
13. DNS reachable
14. Target DNS addresses reachable
15. Management VLAN assigned + available

**After Installing AP:**
1. Document AP MAC + assigned name
2. Document AP location
3. Document AP switch + port
4. Document AP IP address
5. Confirm AP installed in proper orientation
6. Confirm external antennas installed correctly
7. Wait for AP to receive configuration
8. Wait for 2nd reboot if needed
9. Listen for all SSIDs broadcast
10. Connect client to each SSID
11. Check each SSID for proper VLAN + IP pool
Then run Wi-Fi Connection Testing Checklist (14 steps above)

---

## SHEET 5: Passpoint v4 (30 Configuration Items)
SSID name, security (WPA2/3 Enterprise), 802.11r, Hotspot 2.0 selected, RADIUS auth/accounting (IP, port, shared secret ×2), RadSec (IP/domain + port), Interim Update Interval, Venue Name/Type, Network Type, IP Type Availability, NAI Realm (name, EAP method, EAP sub-methods), Roaming Consortium (name, OI), 3GPP Cellular Networks, PMLN IDs (MCC+MNC), NAS ID

Passpoint enabled validation: Interworking IE, ANQP, Roaming Consortium IE, RCOIs (×3), Wi-Fi Alliance IE, Hotspot 2.0 Type 16, security/cipher/AKM, Beacon mode, certificates on clients, client connect test

---

## SHEET 6: Wi-Fi 7 v4 (19 MLO Checklist Items)
1. SSID name correct
2. SSID across required bands
3. MLO Enabled
4. MLD MAC present per BSSID
5. EMLSR Support
6. EMLMR Support
7. Link ID present
8. Max simultaneous links
9. Security type consistent
10. AKM Suite Type
11. Group Cipher Suite Type
12. Pairwise Cipher Suite Type
13. Reduced Neighbor Report (RNR) present
14. RNR Channels
15. Same SSID
16. Transmitted BSSID
17. RNR BSSIDs
18. MFP Required
19. MFP Capable

---

## SHEET 7: eduroam (same structure as Passpoint v4 + eduroam validation)
Identical to Passpoint checklist but for eduroam network configuration.

---

## SHEET 8: Apple Best Practices
- Unique SSID name (avoid conflicts with nearby networks)
- NOT hidden (no security benefit, causes performance issues)
- WPA3 Personal (or WPA2/WPA3 transition as fallback)
- Strong password
- Use 5GHz (more bandwidth, less interference than 2.4GHz)
- Channel = Automatic (let router select best channel)
- Fix Preferred Network Mismatch: Forget + re-join if conflicting info
