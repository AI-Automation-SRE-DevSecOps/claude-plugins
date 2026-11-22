# Alerting on SLOs — SRE Workbook, Chapter 5

Source: https://sre.google/workbook/alerting-on-slos/ (Steven Thurgood with Jess Frame, Anthony Lenton, Carmela Quinito, Anton Tolchanov, Nejc Trdin). Examples use a request-driven service and Prometheus syntax, but the approach applies to any alerting framework.

## Core principles

- Goal: be notified of a **significant event**, defined as "an event that consumes a large fraction of the error budget" (Alerting Considerations).
- "Error budget" and "error rate" apply to all SLIs. SLIs are good events / total events. The error budget is the number of allowed bad events. The error rate is bad / total.
- Judge an alerting strategy on four attributes (Alerting Considerations):
  - **Precision**: the proportion of detected events that were significant. Low traffic makes alerts more sensitive to nonsignificant events.
  - **Recall**: the proportion of significant events that were detected.
  - **Detection time**: how long it takes to notify. Long detection times eat budget.
  - **Reset time**: how long alerts keep firing after the issue is resolved. Long reset times cause confusion or get ignored.
- **Burn rate** is how fast, relative to the SLO, the service consumes error budget. Burn rate 1 leaves exactly 0 budget at the end of the SLO window. For a 99.9%/30-day SLO, a steady 0.1% error rate is burn rate 1 (Approach 4).
- Pages and tickets are the only valid ways to get a human to act (footnote 2).
- Conclusion: multiwindow, multi-burn-rate alerting is "the most appropriate approach" in most cases.

## Formulas (as written)

- Approach 1, budget spend detected when alerting at error rate = SLO: `alerting window size / reporting period`
- Approach 2, detection time: `((1 − SLO) / error ratio) × alerting window size`
- Approach 4, time for a burn-rate alert to fire: `((1 − SLO) / error ratio) × alerting window size × burn rate`
- Approach 4, budget consumed when the alert fires: `(burn rate × alerting window size) / period`
- Recording rule: `job:slo_errors_per_request:ratio_rate10m = sum(rate(slo_errors[10m])) by (job) / sum(rate(slo_requests[10m])) by (job)`. If those metrics aren't exported, create them by renaming (`record: slo_errors`, `expr: http_errors`).

## The six approaches (Ways to Alert on Significant Events)

Approaches 1–3 don't work. Approaches 4–6 do, and 6 is the one Google recommends most strongly.

| # | Approach | Example rule (99.9% / 30d) | Pros | Cons |
|---|---|---|---|---|
| 1 | Target error rate ≥ SLO threshold | `ratio_rate10m >= 0.001` | Detection time 0.6 s for a total outage. Good recall. | Low precision. 0.1% errors for 10 min alerts but uses only 0.02% of the monthly budget. Up to 144 alerts/day with no action taken, and you'd still meet the SLO. |
| 2 | Increased alert window (5% of the 30-day budget = 36h window) | `ratio_rate36h > 0.001` | Detection 2 min 10 s for a complete outage. Better precision. | Very poor reset time: fires for 36 h after a 100% outage. Long windows are expensive in memory and I/O. |
| 3 | Incrementing alert duration | `ratio_rate1m > 0.001`, `for: 1h` | Can be higher precision. | Poor recall and detection. A 100% outage alerts after 1 h, the same as a 0.2% outage, and burns 140% of the 30-day budget in that hour. The timer resets if the metric briefly recovers. 5-min 100% spikes every 10 min never alert yet burn 35% of the budget (each spike almost 12%). **Not recommended.** |
| 4 | Alert on burn rate (1h window, 5% spend gives burn rate 36) | `ratio_rate1h > 36 * 0.001` | Good precision. Shorter window is cheaper to compute. Good detection time. Reset time 58 min. | Low recall: a 35x burn never alerts but uses the whole 30-day budget in 20.5 h. A 58-min reset is still too long. |
| 5 | Multiple burn-rate alerts | page: `1h > 14.4*0.001 or 6h > 6*0.001`; ticket: `3d > 0.001` | Adapts to criticality. Good precision. Good recall (3-day window). Lets you pick page vs ticket. | More numbers to manage. Even longer reset (3d). Needs alert suppression, because 10% spent in 5 min also trips the 6h and 1h conditions, giving 3 notifications. |
| 6 | **Multiwindow, multi-burn-rate** | each long window ANDed with a short window (below) | Flexible. Good precision. Good recall. | Many parameters, hard to manage (see Alerting at Scale). |

Footnote 1: duration clauses can occasionally help filter ephemeral noise over very short durations, but the cons above still apply.

### Table 5-4: Burn rates and time to complete budget exhaustion (99.9% SLO)

| Burn rate | Error rate | Time to exhaustion |
|---|---|---|
| 1 | 0.1% | 30 days |
| 2 | 0.2% | 15 days |
| 10 | 1% | 3 days |
| 1,000 | 100% | 43 minutes |

### Table 5-6: Multiple burn rates (Approach 5)

| Budget consumption | Time window | Burn rate | Notification |
|---|---|---|---|
| 2% | 1 hour | 14.4 | Page |
| 5% | 6 hours | 6 | Page |
| 10% | 3 days | 1 | Ticket |

These are "reasonable starting numbers". The right values depend on the service and baseline page load. Busier services, or teams with weekend and holiday on-call concerns, may make the 6h window a ticket instead.

### Table 5-8: Recommended parameters for a 99.9% SLO (Approach 6)

| Severity | Long window | Short window | Burn rate | Error budget consumed |
|---|---|---|---|---|
| Page | 1 hour | 5 minutes | 14.4 | 2% |
| Page | 6 hours | 30 minutes | 6 | 5% |
| Ticket | 3 days | 6 hours | 1 | 10% |

- Short window = **1/12 of the long window**. Example (Fig 5-6): with 15% errors for 10 min, the alert starts firing 5 min in, when the long window crosses the threshold. It stops 5 min after the errors end, when the short window drops back. The long window alone would take 60 min to drop.
- Page rule: `(1h > 14.4*0.001 and 5m > 14.4*0.001) or (6h > 6*0.001 and 30m > 6*0.001)`.
- Ticket rule: `(24h > 3*0.001 and 2h > 3*0.001) or (3d > 0.001 and 6h > 0.001)`. The example config includes a **24h/2h at 3x** ticket condition that is not listed in Table 5-8.
- The thresholds are burn rate × (1 − SLO), e.g. 14.4 × 0.001.

## Low-Traffic Services and Error Budget Alerting

- Example: 10 req/hour means 1 failure is a 10% hourly error rate. Against 99.9% that is a 1,000x burn rate, uses 13.9% of the 30-day budget, and pages immediately. Only 7 failed requests are allowed in 30 days.
- Options: **generate artificial traffic**, **combine services**, or **modify the product** so that more requests are needed to count a failure, or a single failure has lower impact.
  - *Artificial traffic*: reuses existing monitoring and SLOs (probers, integration tests). Downsides: it covers only a small portion of request types, which is worse for stateful services. Successful synthetic requests can also hide real-user errors.
  - *Combining services*: the services must be related (same product, or request types in the same binary). One service's complete failure may not register, so choose a shared failure domain (e.g., a common DB) and keep longer-period alerts for individual 100% failures.
  - *Service/infra changes*: client retries with exponential backoff and jitter, and fallback paths that capture requests for later execution.
  - *Lowering the SLO or increasing the window*: for example 99.9% to 99%, negotiated with stakeholders. Downside: it is a product decision, and it affects expectations and when the error budget policy is enacted.
- In practice Google combines fake traffic (where it gives good coverage), client changes, aggregation of services that share a failure mode, and SLO thresholds commensurate with the actual impact.

## Extreme Availability Goals

- A 90% SLO: a 100% outage uses only 1.4% of the budget per hour, so the 2%-in-1h page **can never fire**. Tune the parameters.
- A 99.999% monthly SLO: a 100% outage exhausts the budget in **26 seconds**. That is less than many metric collection intervals, so alerting can't defend it. Design so that a 100% outage is very unlikely, for example by rolling out to 1% of users first, which gives 43 minutes (see Canarying Releases).

## Alerting at Scale

- **Anti-pattern**: per-service custom window and burn-rate parameters, for example across 100 microservices or request types. It creates toil and cognitive load that doesn't scale. Decide the parameters once and apply them to all services. The only exception is temporary changes during an ongoing outage (footnote 5).
- Group request types into buckets (Table 5-10):

| Class | Availability | Latency @ 90% | Latency @ 99% |
|---|---|---|---|
| CRITICAL (e.g., login) | 99.99% | 100 ms | 200 ms |
| HIGH_FAST (core interactive) | 99.9% | 100 ms | 200 ms |
| HIGH_SLOW (e.g., multi-year report) | 99.9% | 1,000 ms | 5,000 ms |
| LOW (e.g., notification polling) | 99% | None | None |
| NO_SLO (dark launches, alpha) | None | None | None |

"Latency @ 90%" means 90% of requests are faster than the threshold.

## Anti-patterns summary

- Alerting on short-window error rate ≥ SLO, which gives a flood of non-actionable alerts (Approach 1).
- A single long window alone, which gives poor reset time and is expensive (Approach 2).
- `for:` duration clauses as a substitute for windows (Approach 3).
- A single burn rate, which misses slower burns such as 35x (Approach 4).
- Multiple burn rates without suppression, which sends duplicate notifications (Approach 5).
- Per-service hand-tuned parameters (Alerting at Scale).
- Applying the 99.9% defaults unchanged to very low or very high SLOs (Extreme Availability Goals).

## Checklist

1. Define SLIs as good/total ratios, plus the SLO and window (e.g., 99.9% / 30d).
2. Create recording rules for error ratios over 5m, 30m, 1h, 2h, 6h, 24h, and 3d.
3. Page on 14.4x (1h AND 5m) OR 6x (6h AND 30m). Ticket on 1x (3d AND 6h), and optionally 3x (24h AND 2h).
4. Check traffic volume. If it is low, use synthetic traffic, aggregation, retries or fallbacks, or adjust the SLO or window.
5. Check SLO extremes (≤90% or ≥99.99%) and re-derive the parameters.
6. Bucket request classes and apply the same parameters to every bucket.

## Quotes

- "an event that consumes a large fraction of the error budget."
- "we do not recommend using durations as part of your SLO-based alerting criteria."
- "A good guideline is to make the short window 1/12 the duration of the long window"
- "pages and tickets are the only valid ways to get a human to take action."
- "we strongly advise against specifying the alert window and burn rate parameters independently for each service"
