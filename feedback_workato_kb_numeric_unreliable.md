---
name: feedback_workato_kb_numeric_unreliable
description: Workato KB Search cannot be trusted for exact numeric table cells (benchmarks, TPS tables) even when it names the right source document — use direct WebFetch of the primary source instead
type: feedback
originSessionId: 7403a6e9-ad49-464b-9285-c295b912c840
---

For any exact numeric benchmark/table figure (RADIUS TPS, throughput, sizing tables, etc.), never rely on Workato KB Search's semantic retrieval to quote the numbers — always independently verify via a direct `WebFetch` of the primary document/page.

**Why:** Discovered 2026-09-12 during EN_NAC-CE vs Cisco ISE benchmark work. A separate session queried the same named Cisco source ("Performance and Scalability Guide for Cisco ISE," Table 5) twice via Workato KB Search and got materially different, internally inconsistent numbers each time (e.g. one query had LDAP-backed auth faster than AD-backed auth, the other had the reverse). This session then independently re-fetched the actual live Cisco page directly via `WebFetch` and it matched the *original* (correct) numbers exactly. Conclusion: Workato KB Search's retrieval/summarization over indexed PDFs scrambles rows/columns of numeric tables even while correctly identifying the source document by name — it is fine for locating documents and summarizing qualitative content, but not for extracting precise numeric cells.

**How to apply:** When Workato KB Search (or any KB/semantic-search connector) surfaces a document containing hard numbers a user will rely on (competitive benchmarks, sizing tables, capacity figures, client-facing claims), always follow up with a direct WebFetch/WebSearch of the primary source before quoting the numbers — especially for anything going in front of a client or into a permanent competitive-positioning document. Flag clearly in the document which numbers came from which method (KB search vs. direct fetch) so future readers know the confidence level.
