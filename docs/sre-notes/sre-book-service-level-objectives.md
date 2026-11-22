# SRE Book, Chapter 4: Service Level Objectives

Source: https://sre.google/sre-book/service-level-objectives/
Authors: Jones, Wilkes, Murphy, with Smith.

**Scope note:** The *error budget* gets only a brief mention here, with a pointer to Ch. 3. There is **no** burn-rate alerting or error-budget policy in this chapter; those come from the SRE Workbook.

## Core principles

- **The three terms are separate** (*Service Level Terminology*):
  - **SLI**: "a carefully defined quantitative measure of some aspect of the level of service that is provided."
  - **SLO**: "a target value or range of values for a service level that is measured by an SLI."
  - **SLA**: "an explicit or implicit contract with your users that includes consequences of meeting (or missing) the SLOs they contain."
- **How to tell an SLO from an SLA:** ask "what happens if the SLOs aren't met?" If there is no explicit consequence, it is almost certainly an SLO. Footnote 16: people who say "SLA violation" almost always mean a missed SLO.
- **SRE doesn't usually build SLAs.** Business and legal teams own them. SRE helps avoid triggering their consequences, helps define objective SLIs, and advises on how likely and how hard the SLOs are to meet (*Agreements*, *Agreements in Practice*).
- **Define SLIs and SLOs even without an SLA** (for example, Google Search has no public SLA).
- **Start from what users care about, not from what is easy to measure.** It can work better to start from the desired objectives and work back to indicators (*Objectives in Practice*).
- **100% is the wrong target.** Insisting SLOs are met 100% of the time slows innovation and deployment and/or needs expensive, overly conservative solutions. Allow an error budget instead.
- **SLOs drive priorities.** They are "a massive lever" and "a helpful, legitimate forcing function for a development team."

## Concrete numbers, thresholds and formulas (as written)

- **SLO structure:** `SLI ≤ target`, or `lower bound ≤ SLI ≤ upper bound`.
- **"Nines":** 99% = "2 nines"; 99.999% = "5 nines". The published Google Compute Engine target is "three and a half nines", which is **99.95%**.
- **Availability** = the fraction of the time a service is usable, often defined as the fraction of well-formed requests that succeed. This is sometimes called **yield**. **Durability** = the likelihood that data will be kept over a long period.
- **Example SLO:** average Shakespeare search latency < **100 ms**. The chapter calls 100 ms "obviously an arbitrary value".
- **Aggregation example:** 200 req/s in even seconds and 0 in odd seconds averages to 100 req/s, but the instantaneous load is twice that.
- **Figure 4-1:** shows the 50th, 85th, 95th and 99th percentile latencies on a log Y-axis. A typical request takes about **50 ms**, but **5% of requests are 20× slower**. Average-only monitoring would show no change over the day.
- **Percentiles:** the 99th or 99.9th percentile shows a plausible worst case, and the 50th (median) shows the typical case.
- **Skew example:** no response can take less than 0 ms, and a **1,000 ms** timeout caps successful responses. So the mean and the median can be far apart.
- **Example SLO specs:**
  - "99% (averaged over 1 minute) of Get RPC calls will complete in less than 100 ms (measured across all the backend servers)."
  - Tiered targets: 90% < 1 ms; 99% < 10 ms; 99.9% < 100 ms (Get RPCs).
  - Per workload class: 95% of throughput clients' Set RPCs < 1 s; 99% of latency clients' Set RPCs with payloads < 1 kB < 10 ms.
- **Error budget:** "a rate at which the SLOs can be missed". Track it **daily or weekly**. Upper management will probably also want a **monthly or quarterly** assessment. The gap between the SLO violation rate and the budget is an input to release decisions.

## Recommended practices and procedures

- **Pick a handful of SLIs by system type** (*What Do You and Your Users Care About?*):
  - Serving systems: availability, latency, throughput.
  - Storage: latency, availability, durability.
  - Pipelines: throughput, end-to-end latency.
  - All systems: correctness, though it is usually not SRE's to meet.
- **Collect on the client side where it matters** (*Collecting Indicators*). Server-side metrics can miss problems users feel, such as slow JavaScript.
- **Use percentiles and don't assume normality without checking** (*Aggregation*, *A Note on Statistical Fallacies*).
- **Standardize SLIs with reusable templates** (*Standardize Indicators*).
- **Keep a safety margin.** Use a tighter internal SLO than the one you advertise (*SLOs Set Expectations*).
- **Don't overachieve.** Prevent over-dependence with planned outages, throttling some requests, or a design that isn't faster under light load.
- **Run the control loop** (*Control Measures*):
  1. Monitor and measure the SLIs.
  2. Compare the SLIs to the SLOs and decide whether action is needed.
  3. If action is needed, work out what must happen to meet the target.
  4. Take that action.
- **Publish SLOs** to avoid both over-reliance and under-reliance. **Be conservative in SLAs**: the broader the audience, the harder they are to change.

## The Chubby case (sidebar *The Global Chubby Planned Outage*)

Global Chubby was so reliable that services came to depend on it, so its rare outages became user-visible. SRE now keeps it at a level that "meets, but does not significantly exceed" its SLO. In any quarter where real failures haven't pushed availability below target, SRE causes a controlled outage.

## Anti-patterns Google warns against (*Choosing Targets* and elsewhere)

- **Picking a target based on current performance.** This can lock you into heroic efforts or a redesign.
- **Complicated aggregations.** They hide changes in performance and are harder to reason about. Keep it simple.
- **Absolutes** such as "infinite" scale or "always" available. They are unrealistic, expensive, and better than users need.
- **Too many SLOs.** "If you can't ever win a conversation about priorities by quoting a particular SLO, it's probably not worth having that SLO."
- **Overly strict initial targets.** Start loose and tighten later; it is better than relaxing a target you can't meet. *Perfection can wait*.
- **Every trackable metric as an SLI**; **averages only**; and **overachieving** the SLO (see above).

## Templates and checklists

**SLI standard-definition defaults.** If an SLI conforms to these defaults, they can be omitted from its spec:

| Attribute | Example default |
|---|---|
| Aggregation intervals | "Averaged over 1 minute" |
| Aggregation regions | "All the tasks in a cluster" |
| Measurement frequency | "Every 10 seconds" |
| Requests included | "HTTP GETs from black-box monitoring jobs" |
| Data acquisition | "Through our monitoring, measured at the server" |
| Data-access latency | "Time to last byte" |

**SLO spec checklist:** state the percentage, the operation, the threshold, the aggregation window, where it is measured, and the workload class.

## Short verbatim quotes

- "Start by thinking about (or finding out!) what your users care about, not what you can measure."
- "An error budget is just an SLO for meeting other SLOs!"
- "Most metrics are better thought of as distributions rather than averages."
- "Users build on the reality of what you offer, rather than what you say you'll supply"
- "It's better to start with a loose target that you tighten than to choose an overly strict target"
- "SLOs are a massive lever: use them wisely."
