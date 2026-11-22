# SLIs and SLOs

Sources: SRE Book ch.3 "Embracing Risk", ch.4 "Service Level Objectives", App. A "Availability Table". SRE Workbook ch.2 "Implementing SLOs", ch.3 "SLO Engineering Case Studies", ch.19 "Reaching Beyond Your Walls", App. A "Example SLO Document".

## Definitions

| Term | Meaning |
|---|---|
| **SLI** (indicator) | A measure of some aspect of service level. Express it as `good events / valid events` on a 0–100% scale |
| **SLO** (objective) | A target for an SLI over a window: `SLI ≥ target`. If missing it has no explicit consequence, it's an SLO, not an SLA |
| **SLA** (agreement) | A business contract with consequences (refunds, credits). SRE helps avoid missing SLAs but usually doesn't write them. Keep the SLO stricter than the SLA |
| **Error budget** | `1 − SLO`: the amount of unreliability you are allowed to spend |
| **CUJ** (critical user journey) | What a user is trying to do, such as log in, check out or load the feed. SLIs should measure CUJs |

## Principles

- **100% is the wrong target** for almost everything. Users can't tell 99.999% from 100% because their own network and device are less reliable, typically failing 0.01–1% of the time. Each step up can cost up to 100× more (redundant capacity plus opportunity cost), and it blocks change, which is the main source of outages. (Book ch.3)
- **The target is a product decision.** Product, development and SRE agree on it. An "empowered owner" holds it (the CTO or the product owner). (Workbook ch.2)
- **Don't overachieve by much.** If a service is far more reliable than its SLO, users start depending on that. Google's Chubby team plans controlled outages when real availability is well above the target. (Book ch.4)
- **Four prerequisites before an SLO counts** (Workbook ch.2):
  1. Stakeholders approve it.
  2. The people defending it agree it is achievable.
  3. Everyone commits to using the budget for decisions, written down as an error budget policy.
  4. There is a process for refining it.
- **The first SLO doesn't have to be right.** Start somewhere and iterate. Review it about monthly at first, then quarterly.
- **Start loose and tighten.** It is easier to tighten an SLO later than to loosen one users already rely on. (Book ch.4)

## The SLI menu (Workbook ch.2, Table 2-1)

| Component | SLI types |
|---|---|
| Request-driven (API, web) | **Availability** (share of requests served successfully), **Latency** (share served faster than a threshold), **Quality** (share served without degradation) |
| Pipeline (batch, streaming) | **Freshness** (share of data updated more recently than a threshold), **Correctness** (share of output that is correct), **Coverage** (share of data processed) |
| Storage | **Durability** (share of written data that can be read back) |

Use **five or fewer SLI types** per service. For latency, use two or more thresholds to capture the tail, for example "90% < 100 ms, 99% < 400 ms".

## Specification vs implementation

- **SLI specification**: the user outcome in words, for example "the proportion of home page requests that load in under 100 ms".
- **SLI implementation**: how you measure it. Each option trades off quality, coverage and cost:

| Measured at | Pros | Cons |
|---|---|---|
| Application server metrics or logs | Cheap, detailed | Misses requests that never reach the server |
| **Load balancer metrics** | Close to the user, cheap, covers whole-server failures | Misses client-side and network problems |
| Synthetic probers (black-box) | Covers paths with no traffic, needed for low-traffic services | Covers only the scripted paths; can hide real-user errors |
| Client instrumentation (RUM) | Closest to the user | Expensive and noisy; outside your control |

Start with the cheapest implementation that's good enough, often load balancer metrics. Move closer to the user if the SLI misses user-visible problems (low recall).

## Defining "good", "bad" and "excluded"

Write these down explicitly in the SLO document.

- **Availability, typical default** (Workbook ch.2 and App. A): any status outside 500–599 is good. That counts 4xx as success, because a 4xx is usually the client's fault.
  - **Exceptions to check**: `429` from your own rate limiting, and `499` or client timeouts caused by your latency, are user-visible failures. Either count them as bad or justify excluding them.
  - Never return an error inside a `200` body. Home Depot's rule is that only 5xx counts toward the SLO, so errors must use the correct status code. (Workbook ch.3)
  - Implicit errors (a `200` with wrong content) are only caught by end-to-end probes or correctness checks.
- **Exclude from valid events**: health checks, synthetic and load-test traffic (unless the prober is the SLI), canary traffic when it is judged separately, and drained regions.
- **Latency**: count failed requests as bad, or measure latency only for successful ones, but never let fast errors make latency look good. "A slow error is even worse than a fast error." (Book ch.6)

## Windows

- **Default: a four-week rolling window** (Workbook ch.2, App. A/B). Whole weeks mean every window contains the same number of weekends, which a 30-day window does not.
- Calendar windows (monthly or quarterly) line up with business reporting and SLAs. Evernote deliberately used a calendar month. (Workbook ch.3)
- Add weekly summaries for prioritizing work and quarterly reports for planning.
- Error budgets only approximate user happiness. One 4-hour outage, four 1-hour outages and a constant 0.5% error rate can cost the same budget but feel very different to users.

## Choosing targets

- **Starter SLOs can come from historical performance**, as long as you iterate (Workbook ch.2, App. A).
  - Round availability **down** to two significant figures, or to the nearest 1% for a new service.
  - Round latency **up** to the nearest 50 ms, because users rarely notice 50 ms.
  - Example: measured 97.123% success, p90 432 ms and p99 891 ms. The SLOs become 97%, 90% < 450 ms and 99% < 900 ms.
- Then validate against users. Correlate support tickets, complaints and outages with budget burn, and tighten or loosen accordingly.
- **Table 2-5 decision matrix** (Workbook ch.2):

| SLO | Toil | Customer satisfaction | Action |
|---|---|---|---|
| Met | Low | High | Relax release processes and ship faster, or step back from the engagement |
| Met | Low | Low | Tighten the SLO |
| Met | High | High | Fix false-positive alerts, or loosen the SLO temporarily or offload toil, then fix the product |
| Met | High | Low | Tighten the SLO |
| Missed | Low | High | Loosen the SLO |
| Missed | Low | Low | Make alerting more sensitive |
| Missed | High | High | Loosen the SLO |
| Missed | High | Low | Offload toil and fix the product or its automated mitigation |

- **Aspirational SLOs** are tracked but don't trigger the policy. Use them when you can't yet meet a target you want.
- **Bucket request classes** rather than hand-tuning each endpoint (Workbook ch.5, Table 5-10):

| Class | Availability | 90% under | 99% under |
|---|---|---|---|
| CRITICAL (e.g. login) | 99.99% | 100 ms | 200 ms |
| HIGH_FAST (core interactive) | 99.9% | 100 ms | 200 ms |
| HIGH_SLOW (e.g. large reports) | 99.9% | 1,000 ms | 5,000 ms |
| LOW (e.g. notification polling) | 99% | none | none |
| NO_SLO (dark launch, alpha) | none | none | none |

These are Google's illustrative buckets, not universal defaults. Agree your own with stakeholders.

## Dependencies and composition

- **Availability multiplies across serial hard dependencies.** A service at 99% on a platform at 99.999% gets at best 98.99901%. (Workbook ch.19)
- Each critical dependency with availability P lowers yours by roughly `1 − P`. A four-nines service with nine four-nines dependencies is effectively three nines. (Workbook ch.6)
- **Redundant zones are not independent.** Two 99.9% zones don't give 99.9999%, because they share dependencies and control planes. (Workbook ch.2)
- A critical dependency's guarantee must be at least as high as the action that relies on it. Otherwise make it a soft dependency (degrade, cache, fall back).
- **Customers without a stated SLO invent one** and tell you only when you miss it. Publish SLOs, help them alert on theirs, and build shared dashboards. (Workbook ch.19)

## Availability table (Book App. A)

Assumes no planned downtime, so maintenance counts against the budget. "Month" means 30 days here. For a 4-week window, multiply the budget fraction by 40,320 minutes.

| Availability | Per year | Per 30 days | Per 28 days | Per week | Per day |
|---|---|---|---|---|---|
| 99% | 3.65 days | 7.2 hours | 6.72 hours | 1.68 hours | 14.4 min |
| 99.5% | 1.83 days | 3.6 hours | 3.36 hours | 50.4 min | 7.2 min |
| 99.9% | 8.76 hours | 43.2 min | 40.3 min | 10.1 min | 1.44 min |
| 99.95% | 4.38 hours | 21.6 min | 20.2 min | 5.04 min | 43.2 s |
| 99.99% | 52.6 min | 4.32 min | 4.03 min | 1.01 min | 8.64 s |
| 99.999% | 5.26 min | 25.9 s | 24.2 s | 6.05 s | 0.86 s |

Prefer **request-based** availability (successful / total requests) over time-based uptime. It handles partial outages and changing load better. (Book ch.3)

## Low-traffic services (Workbook ch.5)

At 10 requests per hour, one failure is a 10% error rate, a 1,000× burn on a 99.9% SLO. Options:
- Generate synthetic traffic.
- Combine related services that share a failure domain into one SLO.
- Change the product so that one failure matters less (retries with backoff, queue for later).
- Lower the SLO or lengthen the window, as a product decision.

## SLI implementation examples (Prometheus)

```promql
# Availability: anything not 5xx is good; health checks excluded
sum(rate(http_requests_total{job="api", code!~"5..", handler!~"/healthz"}[28d]))
  / sum(rate(http_requests_total{job="api", handler!~"/healthz"}[28d]))

# Latency: share of successful requests under 450 ms. Needs a histogram bucket at le="0.45".
sum(rate(http_request_duration_seconds_bucket{job="api", le="0.45", code!~"5.."}[28d]))
  / sum(rate(http_request_duration_seconds_count{job="api", code!~"5.."}[28d]))
```

Notes:
- Raw `[28d]` ranges are expensive. In production, build them from 5m recording rules, for example `sum_over_time(job:slo_errors:rate5m[28d])`, or use your tool's SLO feature.
- Histogram bucket boundaries must include your SLO thresholds. Otherwise the ratio is interpolated or impossible.
- `histogram_quantile()` gives a percentile **value** for dashboards. It is not a ratio SLI, so don't compare it with a ratio target.

## SLO review checklist

1. Does it measure a CUJ that users notice? Is it measured as close to the user as is affordable?
2. Is the SLI a ratio of good to valid events, with "good", "bad" and "excluded" written down?
3. Does latency use several thresholds, with errors handled so they can't hide slowness?
4. Is the window stated (default 4-week rolling)?
5. Is there a rationale for the target (historical, user research or ad hoc) and is it stated honestly?
6. Is the target below what dependencies can support, and not far below current performance?
7. Have product, dev and SRE approved it? Is there an error budget policy with an escalation path?
8. Is there a next review date?

## SLO document template (from Workbook App. A)

```markdown
# SLO: <service>
Status: Published | Draft · Author(s) · Reviewer(s) · Approver(s)
Approval date · Revisit date

## Service overview
What it does, its users, and its CUJs. Architecture diagram link.

## SLIs and SLOs
| Category | SLI spec | SLI implementation | SLO (4-week rolling) |
|---|---|---|---|
| Availability | Share of requests that don't return 5xx | LB metrics, health checks excluded | 99.9% |
| Latency | Share of requests served under threshold | LB latency histogram | 90% < 200 ms; 99% < 1,000 ms |
| Freshness (pipeline) | Share of reads using data updated < 10 min ago | Read-time metric | 99% |

## Rationale
How the targets were chosen (for example, 4 weeks of data rounded down to the nearest 1%; latency rounded up to 50 ms).
Say openly if the targets aren't yet validated against user experience.

## Error budget
Budget per SLO = 100% − SLO, tracked separately for each objective. The policy applies when any budget is exhausted.
Link to the error budget policy.

## Clarifications and caveats
Excluded traffic, how status codes are classified, known measurement gaps.
```
