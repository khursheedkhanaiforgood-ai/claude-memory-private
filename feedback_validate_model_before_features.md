---
name: Validate simulator physics before building features on top
description: Don't add Monte Carlo, optimizer, or UI features to a simulator with broken physics
type: feedback
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
June 3: full physics audit revealed 17 bugs in stadium_wifi_simulator_v2.html. The multiplier chain produces 46,000 Gbps capacity and 2% airtime — both physically impossible. All MC and optimizer outputs built on top of this are meaningless.

**Why:** User asked "the outputs don't make sense" — and they were right. Building Monte Carlo confidence bands on a model that is off by 20× is worse than no model.

**How to apply:** For ANY simulator/model work:
1. Validate core physics first (airtime, capacity, goodput all consistent)
2. Run a sanity check: at full stadium load (72K fans, 55% concurrency), airtime should be 20–60%, NOT 2%
3. Only build stochastic/optimizer layers once deterministic layer is correct
4. Check: does muF multiply a rate that already includes spatial streams? Does capMult have a physical ceiling?
