---
name: Wired→wireless unicast diagnostic chain
description: When clients have DHCP but no internet and SW1 routing is confirmed healthy, use this 4-step chain to diagnose AP data plane unicast delivery failure.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---

When clients have DHCP IPs but cannot reach internet (or even their own gateway), and SW1 routing/QF-Modem routing is confirmed healthy, the break is often **AP not delivering unicast wired→wireless frames**.

**Diagnostic chain (in order):**

1. `show fdb vlan <vlan>` on SW1 — client MACs learned on correct port? If yes, wireless→wired works.
2. `show iparp vlan <vlan>` on SW1 — SW1 has ARP entries? (Populated by gratuitous ARPs from clients, which are broadcasts — even if unicast is broken.)
3. `ping vr VR-Default <client_ip>` from SW1 — 100% loss despite valid ARP = wired→wireless unicast broken at AP.
4. `arp -an` on client MacBook — **empty ARP cache = smoking gun**. Client's ARP request for gateway went out (wireless→wired works) but SW1's unicast ARP reply never arrived (wired→wireless broken at AP).

**macOS ARP command syntax (arp -n alone returns nothing on macOS — always use -a):**
```
arp -an                # show full cache, numeric — USE THIS for diagnostics
arp -a                 # show full cache with hostname resolution
arp -n <ip>            # lookup one IP, no resolution
arp -d <ip>            # delete one entry
sudo arp -a -d         # flush entire ARP cache
```

**Flush-and-retest procedure (cleanest diagnostic):**
```
sudo arp -a -d         # flush cache
ping -c 1 10.x.x.1    # force fresh ARP request
arp -an                # gateway MAC present = unicast working ✅ / empty = broken ❌
```

**Healthy ARP output after AP reboot (example — MacBook on VLAN 100):**
```
10.100.0.1      → <SW1 MAC>    ← gateway resolved = unicast working ✅
10.100.0.255    → ff:ff:ff:ff  ← subnet broadcast (normal)
224.0.0.251     → multicast    ← mDNS/Bonjour (normal, macOS always present)
255.255.255.255 → broadcast    ← DHCP broadcast (normal)
```
Note: 8.8.8.8 and external IPs will NEVER appear in arp -an — all external traffic goes via the gateway MAC.

**Why DHCP works but unicast doesn't:** iPhones and Macs set the BROADCAST flag in DHCP DISCOVER, so DHCP OFFER/ACK are sent as broadcasts (not unicast). Broadcasts still work even when AP unicast delivery is broken.

**Fix:** XIQ-triggered AP reboot. See feedback_ap_reboot_xiq.md.

**Key EXOS note:** `ping vr VR-Default <ip> from <src>` — the `from` address MUST be one of SW1's own configured IPs (interface IPs). Client IPs are rejected with "Invalid from address supplied." This only tests SW1's own traffic, NOT client-forwarded traffic.
