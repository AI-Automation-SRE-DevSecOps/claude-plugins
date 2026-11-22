# SRE Workbook Ch. 12 — Introducing Non-Abstract Large System Design (NALSD)

Source: https://sre.google/workbook/non-abstract-design/ (Virji, Youngman, Robertson, Thorne, Rensin, Egyed, with Bondi)

## Core principles
- **Reliability is the most critical feature** of any production system; deferring reliability during design "is akin to accepting fewer features at higher costs" (intro).
- **NALSD** = iterative design: start from problem statement, gather requirements, iterate through increasingly sophisticated designs until viable. Combines capacity planning, component isolation, and graceful degradation (*What Is NALSD?*).
- **"Non-abstract"**: every system runs on real computers, datacenters, networks; designers must repeatedly turn a whiteboard design into concrete resource estimates at multiple steps (*Why "Non-Abstract"?*).
- **Reasoning > final numbers**: perfect assumptions aren't required; value lies in combining many imperfect-but-reasonable results.
- Separate components by how you expect the system to grow, so parts scale independently and no single hardware/software instance is a dependency (*Conclusion*).
- NALSD is a learned skill that must be practiced regularly.

## The process (*Design Process*)
Two phases:
1. **Basic design** ("works in principle"): *Is it possible?* (ignoring RAM/CPU/network limits); *Can we do better?* (e.g., O(N) → O(ln(N))).
2. **Scale up** (e.g., dramatically increase a requirement): *Is it feasible?* (money, hardware); *Is it resilient?* (component failure, whole-datacenter failure); *Can we do better?*

Note: the phases list **five** question-slots, but the Conclusion names **"the four key NALSD questions"**: possible ("without 'magic'?"), do better ("as simple as we can reasonably make it?"), feasible ("practical constraints (budget, time, etc.)"), resilient ("survive occasional but inevitable disruptions?"). In practice you bounce between phases and iterate; a design may pass most phases then flounder.

## Worked example: AdWords CTR dashboard — numbers as written
**Requirements (*Initial Requirements*)**
- CTR = clicks / impressions.
- SLOs: **99.9% of dashboard queries complete in < 1 second**; **99.9% of the time, CTR data displayed is less than 5 minutes old**.
- Load: **500,000 search queries/sec**, **10,000 ad clicks/sec**; millions of advertisers.

**One Machine — Calculations**
- Query log entry: time 8 B, query_id 8 B, ad_id "three 64-bit integers, 8 bytes" (sic), search_term up to 500 B, other metadata 500–1,000 B → aggressively rounded to **2 KB/entry**.
- Average CTR **2%** (10,000/500,000), so click log has 2% as many records.
- Use scientific notation to avoid unit errors: (5×10⁵ q/s) × (8.64×10⁴ s/day) × (2×10³ B) = **86.4 TB/day** → rounded to **100 TB/day** (clicks + index overhead).
- 4 TB HDD ≈ **200 IOPS**; 5×10⁵ / 200 = **2,500 disks**; even 10:1 batching → "several hundred HDDs".
- Standard machine: **16 cores, 64 GB RAM, 1 Gbps**. 100 TB / 64 GB = **1,563 machines** for all-RAM.
- Evaluation: one machine has a long list of SPOFs (CPU, memory, storage, power, network, cooling); "even a simple power cycle" breaks SLOs.

**Distributed System — MapReduce**: scales horizontally but batch can't meet the 5-minute freshness SLO; small, nonoverlapping batches miss joins when query and click fall in different batches.

**LogJoiner** (QueryStore keyed by query_id, cf. Bigtable; ClickMap and QueryMap keyed by ad_id). Unmatched clicks are retried up to a time limit, then discarded.
- Click ingest: 10⁴ × 2×10³ B = 20 MB/s = **160 Mbps**
- QueryStore lookup: 10⁴ × 8 B = 80 KB/s = **640 Kbps**; response 20 MB/s = **160 Mbps**
- To ClickMap: 10⁴ × 10³ B = 10 MB/s = **80 Mbps**; aggregate **~400 Mbps**
- ClickMap: 10⁴ × 8.64×10⁴ × 16 B = **14 GB/day** → rounded **20 GB/day** (10M advertisers × 10 ads ≈ 800 MB of ad_ids)
- QueryMap: 3 × 5×10⁵ × 8.64×10⁴ × 16 B = **2 TB/day** (up to three ad_ids per query)

**Sharded LogJoiner** (based on Photon paper). Factors: data management, reliability, efficiency.
- Log sharder: hash query_id, **modulo N, add 1** → shard 1..N; send record there.
- LogJoiner sharded by **query_id**; QueryMap/ClickMap sharded by **ad_id** (known before any read/write).
- Scale horizontally (more instances), not vertically.
- Reliability: sharder sends **duplicate entries to two shards**; place duplicates on different machines; residual risk of double failure covered by **error budget**, with log reprocessing.
- Evaluation: a single datacenter is still a SPOF.

**Multidatacenter**
- Consensus: **three or five replicas**, Paxos, at least one network RTT per write (limits sequential throughput; writes can be partly parallelized).
- Paxos across datacenters "at least a few hundred kilometers apart" ≈ **25 ms/op → 40 sequential ops/sec**; 10⁴ clicks/s → **≥250 processes per datacenter**, sharded by ad_id (add more for backlog/spikes).
- Replication doubles load: 20,000 clicks/s + 1,000,000 queries/s.
- (1.02×10⁶)/40 = **25,500 tasks**; (4×10¹² B)/25,500 = **157 MB/task**; (6.4×10¹⁰)/(1.57×10⁸) = **408 tasks/machine**.
- Network: 1.02×10⁶ × 2 KB = 2.04 GB/s = **16 Gbps**; **640 Kbps/task**; 408 × 640 Kbps = **256 Mbps/machine**.
- Result: **~4 TB RAM per datacenter, 64 machines**, using **25%** of each machine's network bandwidth.

## Recommended practices
- Express requirements as SLOs and check each iteration against them and the error budget.
- Round up aggressively to avoid prematurely hitting limits; use big numbers deliberately.
- Compute storage, IOPS, RAM, and network per component; identify the binding constraint (here, IOPS, then RAM).
- Ask "what happens when this component fails?" for every component, then for the whole datacenter.
- Focus scaling effort on the larger stream (query logs, not clicks).
- Choose shard keys by access pattern; replicate shards and anti-affinitize replicas.
- Delete data too old to be of value.
- Final check: map each requirement to the design feature that satisfies it (*Multidatacenter — Evaluation*).

## Anti-patterns
- Designs that stay on the whiteboard and "don't quite translate in the real world"; last-minute changes for unforeseen physical constraints.
- Deferring reliability to later.
- Single-machine/single-datacenter designs (SPOFs).
- Forcing small batch jobs to meet freshness SLOs where batch boundaries break joins.
- Treating final values as the point instead of the reasoning.

## Checklist (Conclusion's four questions)
1. Is it possible? — without "magic"?
2. Can we do better? — as simple as reasonably possible?
3. Is it feasible? — within budget, time, etc.?
4. Is it resilient? — survives occasional but inevitable disruptions?

## Verbatim quotes
- "We consider reliability to be the most critical feature of any production system."
- "Google has learned (the hard way) that the people designing distributed systems need to develop and continuously exercise the muscle..."
- "Examples of sound reasoning and assumption making are more important than any final values."
- "The value of this exercise is in combining many imperfect-but-reasonable results into a better understanding of the design."
- "NALSD is a learned skill. As with any skill, you need to practice it regularly to maintain your proficiency."
