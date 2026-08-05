---
name: VOSS IQAgent DNS — ip name-server does NOT feed IQAgent
description: In VOSS, ip name-server primary 8.8.8.8 does not populate /etc/resolv.conf for IQAgent. DNS only works if DHCP ran at boot.
type: feedback
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
In VOSS (Fabric Engine), `ip name-server primary 8.8.8.8` configures DNS for the VOSS management plane CLI only. The IQAgent (HiveAgent / cloud agent) is a Linux-level process that reads `/etc/resolv.conf` — which is only populated when DHCP runs at boot time.

**Why:** Confirmed Apr 30 2026 on 5320 running 9.3.2.0.GA. show ip dns showed Status: Inactive / 0 requests even with name-server configured and ping 8.8.8.8 working. IQAgent could not resolve hac.extremecloudiq.com.

**How to apply:** For VOSS XIQ onboarding, NEVER use manual static IP bootstrap and expect IQAgent to resolve DNS. Instead:
1. Connect a port to a DHCP-capable uplink (HomeModem) BEFORE booting
2. Let ZTP+ run at boot — DHCP lease auto-configures /etc/resolv.conf
3. IQAgent picks up DNS from DHCP lease, resolves XIQ, connects
4. Static IP can be set AFTER XIQ is managing the device (via XIQ template)

Also: using the XIQ server IP directly (iqagent server <IP>) fails because TLS cert is issued for the hostname (hac.extremecloudiq.com), not the IP.
