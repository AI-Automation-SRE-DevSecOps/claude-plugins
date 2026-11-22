# The Site Reliability Workbook, Chapter 2: Implementing SLOs

Source: https://sre.google/workbook/implementing-slos/ (Thurgood & Ferguson, with Hidalgo & Beyer)

## Core principles

- **SLOs drive what SREs prioritize** (*Why SREs Need SLOs*).
- **Four prerequisites** (*Getting Started*):
  1. SLOs approved by all stakeholders.
  2. The people defending the SLO agree it can be met under normal circumstances.
  3. A commitment to use the error budget for decisions, formalized in an **error budget policy**.
  4. A process for refining the SLO.
  
  Without these, the SLO is just another KPI.
- **100% is the wrong target** (*Reliability Targets and Error Budgets*). Each extra nine costs more while its value to users approaches zero, and change (the number one source of outages) becomes impossible.
- **An empowered owner holds the SLO:** the CTO in a small organization, the product owner or manager in a large one.
- **An SLA is different.** It is a business contract that triggers compensation (footnote 2).
- **Error budgets only approximate user happiness.** One 4-hour outage, four 1-hour outages and a constant 0.5% error rate cost the same budget (footnote 8).

## Numbers, formulas and tables

- **SLI = good events / total events**, on a 0–100% scale. **Error budget = 100% − SLO.**
  - Example: a 99.9% SLO over 3M requests in 4 weeks allows 3,000 errors. A 1,500-error outage uses 50%.
  - Give every SLI the same shape (numerator, denominator, threshold) so tooling can share inputs.
- **Use "five or fewer" SLI types.**
- **Use multiple latency grades** to catch the long tail: "90% of requests are faster than 100 ms, and 99% of requests are faster than 400 ms."
- **Table 2-1, SLI types by component:**

  | Component | SLI types |
  |---|---|
  | Request-driven | Availability, Latency, Quality |
  | Pipeline | Freshness, Correctness, Coverage |
  | Storage | Durability |

- **Availability:** 5XX responses count against the SLO and everything else is a success.
  `sum(rate(http_requests_total{host="api", status!~"5.."}[7d])) / sum(rate(http_requests_total{host="api"}[7d]))`
  Latency uses `histogram_quantile(0.9 / 0.99, ...[7d])`.
- **Starter SLOs.** Round to "two significant figures of availability, or up to 50 ms of latency." The 50 ms step is used because users rarely notice a 50 ms change.
  - Measured over 4 weeks: 3,663,253 requests, 97.123% success, p90 432 ms, p99 891 ms.
  - **Table 2-3** sets the SLOs: availability 97%, 90% < 450 ms, 99% < 900 ms.
  - **Table 2-4** gives the allowed failures over 4 weeks: 109,897, 366,325 and 36,632.
- **Time window** (*Choosing an Appropriate Time Window*):
  - Default to a **four-week rolling window**.
  - Use whole weeks so every window has the same number of weekends. A 30-day window does not.
  - Add weekly summaries for prioritization and quarterly reports for planning.
- **Incident costing** (*Decision Making*), against the 109,897-error budget:
  - A bad release: 4 hours of NullPointerExceptions, 14,066 errors, 13% of the budget.
  - A database restore: 20 hours, about 72,000 errors, 65%.
  - With 2–3 bad releases per year but one server failure in 5 years, bad pushes cost about twice as much budget, so fix releases first.
- **Table 2-5, decision matrix** (SLO / Toil / Satisfaction → Action):

  | SLO | Toil | Satisfaction | Action |
  |---|---|---|---|
  | Met | Low | High | Relax release processes and increase velocity, or step back from the engagement |
  | Met | Low | Low | Tighten SLO |
  | Met | High | High | Reduce sensitivity if alerts are false positives. Otherwise temporarily loosen the SLO (or offload toil) and fix the product or automated mitigation |
  | Met | High | Low | Tighten SLO |
  | Missed | Low | High | Loosen SLO |
  | Missed | Low | Low | Increase alerting sensitivity |
  | Missed | High | High | Loosen SLO |
  | Missed | High | Low | Offload toil and fix the product or automated mitigation |

- **Bucketing:**
  - By customer tier: Premium 99.99%, Free 99.9%.
  - By responsiveness: interactive requests "90% of requests complete in 100 ms"; CSV downloads "90% of downloads start within 5 s".
- **Recall** is the share of significantly user-impacting events the SLI captures. **Precision** is the share of captured events that really were user-impacting (footnote 7).
- **Critical dependency:** one whose unavailability makes your service unavailable (footnote 9). Its guarantee should be at least as high as the action that depends on it.

## Recommended practices

- **Start simple:**
  1. Pick one application.
  2. Define its users.
  3. List their common and critical interactions.
  4. Draw the architecture.
  5. Classify the components.
- **Separate the SLI specification** (the outcome that matters to users) **from its implementation** (how it is measured). Implementations trade off quality, coverage and cost.
- **Choose the cheapest first implementation,** for example existing logs rather than probers or JavaScript instrumentation. The worked example uses load balancer metrics, which are closer to the user than server logs.
- **Starting from current performance is acceptable** if you have a process for iterating.
- **If there is no data,** set up a simple remote health check (ping or HTTP GET).
- **Stakeholder sign-off:**
  - Product managers agree the threshold is good enough for users.
  - Developers commit to reducing risk when the budget is exhausted.
  - The production team agrees it is defensible "without Herculean effort, excessive toil, and burnout."
- **Policy actions:**
  - Give top priority to reliability bugs from the past four weeks.
  - Work only on reliability until back within SLO.
  - Freeze production changes.
  
  The policy must also name who acts.
- **Review cadence:** review the SLO about monthly at first, then quarterly or less.
- **Continuous improvement:**
  - Correlate outages and support tickets with budget drops (Spearman's rank correlation works).
  - Then tighten or relax the SLO, move the measurement closer to the user or widen coverage, or add an **aspirational SLO** that is tracked but requires no policy action.
  - Iterate, cheapest options first.
- **Emergency mode:** with high-level approval, deprioritize all external demands until the service is back within SLO and the risk of another miss is reduced.
- **When a dependency causes the miss:** the authors favor freezing changes anyway, since that "will make your users happier." Record the choice in the policy.

## Anti-patterns

- An "SLO without teeth," or a 100% goal.
- 30-day windows.
- Tightening the SLO for recall at the cost of precision.
- Enforcing an SLO you can't yet meet. Use an aspirational SLO instead.
- Availability math that assumes independent zones. Two 99.9% zones do not give 99.9999%, because they share dependencies and control planes.
- Durability SLIs that ignore which data users actually want.
- Experiments that degrade the user experience when there is no spare budget, or that misread users who stay only because they have no alternative.

## Templates

**The SLO document includes:**
- Authors, reviewers and approvers
- The approval date and the next review date
- A service description
- The objectives and SLI implementations
- How the budget is calculated and consumed
- The rationale for the numbers (experimental, observational or ad hoc)

**The error budget policy includes:**
- Authors, reviewers and approvers
- Dates
- A service description
- Actions when the budget is exhausted
- An escalation path for disagreements
- Optionally, an overview of error budgets

Full examples are in Appendices A and B.

## Quotes

- "Reliability of 100% is not an engineering culture SLO—it's an operations team SLO."
- "Your first attempt at an SLI and SLO doesn't have to be correct"
- "We have found a four-week rolling window to be a good general-purpose interval."
- "It's pointless to improve the recall of your system if you lower the precision"
