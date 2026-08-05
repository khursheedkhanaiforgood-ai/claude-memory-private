---
name: IQAgent heartbeat log line is not an error
description: The log line "proxy device-connector unknown POST /device-connect/rest/v1/health-check/[serial]" at ~60s cadence is the IQAgent keepalive, not an error. "unknown" is the proxy routing log level, not a connection state.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
When watching XIQ or switch console logs, this line appears every ~60 seconds and is **not an error**:

```
INFO[2026/05/15 17:04:07] proxy device-connector unknown POST
  /device-connect/rest/v1/health-check/FJ012544G-00233
```

**Why it looks alarming:** The word `unknown` appears to indicate an unrecognised device or failed connection. It does not. `unknown` is the log level label for the proxy's catch-all routing handler — it means the request matched a wildcard route, not a named specific route. The HTTP status is 200.

**What it actually means:**
- `FJ012544G-00233` (or similar serial) is the switch's serial number
- The `/health-check/` endpoint is the IQAgent keepalive ping
- ~60 second cadence is normal for the IQAgent heartbeat
- This line CONFIRMS the IQAgent is alive and connected to the XIQ cloud

**How to apply:**
- Seeing this line = IQAgent healthy. No action needed.
- NOT seeing this line for >5 minutes = IQAgent connectivity problem. Check: switch internet access, DNS resolution for XIQ cloud domain, CAPWAP tunnel state.
- Do not file a ticket or reboot the switch because of this log line alone.

**Verified May 15 2026:** SW1 (5320-16P, serial FJ012544G-00233). Line appeared consistently on ~60s cadence throughout the May 15 lab session while IQAgent was confirmed healthy.
