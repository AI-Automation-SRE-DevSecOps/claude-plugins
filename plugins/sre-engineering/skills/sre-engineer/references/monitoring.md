# Monitoring

Sources: SRE Book ch.6 "Monitoring Distributed Systems", ch.10 "Practical Alerting from Time-Series Data", App. B. SRE Workbook ch.4 "Monitoring", ch.3 (VALET).

## Purpose and outputs

Monitoring is for alerting, debugging, long-term trend analysis, before-and-after comparison, and dashboards. It has **three valid outputs** (Book ch.1, App. B):

| Output | Meaning |
|---|---|
| **Page** | A human must act now |
| **Ticket** | A human must act within days |
| **Log / dashboard** | Nobody looks unless they are investigating |

**Email alerts are not an output.** Book App. B calls them "piping them to /dev/null". Replace them with a dashboard of ongoing subcritical problems, or with tickets.

## The four golden signals (Book ch.6)

"If you can only measure four metrics of your user-facing system, focus on these four."

| Signal | What to measure | Pitfalls |
|---|---|---|
| **Latency** | Time to serve a request, as a **distribution** (histogram) | Keep latency of successes and errors separate. Fast 500s improve averages, and "a slow error is even worse than a fast error" |
| **Traffic** | Demand in a unit that fits the system: req/s (split static and dynamic), sessions, transactions or I/O | Track it per request class |
| **Errors** | Rate and ratio of failed requests. They can be **explicit** (5xx), **implicit** (a 200 with wrong content) or **by policy** (slower than the committed latency) | Only end-to-end checks catch wrong content. Alert on a **ratio**, not a raw rate |
| **Saturation** | How full the **most constrained** resource is (CPU, memory, I/O, threads, queue depth, quota), including predictions such as "disk full in 4 h" | Systems degrade before 100%, so a utilization target is needed. Rising latency is often the leading indicator, and p99 over 1 minute gives an early signal |

**VALET** (Home Depot, Workbook ch.3) uses Volume, Availability, Latency, Errors and **Tickets** (manual interventions) as SLO dimensions. Latency SLOs use percentiles, and only 5xx counts toward the error SLO.

## Metrics design (Workbook ch.4, Book ch.10)

- **Use monotonically increasing counters rather than gauges** wherever you can. Counters survive missed scrapes and allow rates over any window up to a month, which burn-rate alerting needs.
- **Sum rates; don't take the rate of a sum.** `sum(rate(x[5m]))` is correct. `rate(sum(x)[5m])` breaks on counter resets.
- **Use histograms for latency, never means.** Bucket boundaries should grow roughly exponentially, by about 3× (10, 30, 100, 300 ms…), and must **include your SLO thresholds**.
- **Use percentiles on dashboards** (p50, p95, p99). Never use a mean for latency, CPU or disk fullness. The p99 of one backend can become the median of the frontend. (Book ch.6)
- **Keep label cardinality bounded.** Status code, method, region and request class are fine. User IDs or request IDs (millions of values) are not.
- **Minimum labels** (Book ch.10): job/service and zone/region, plus the metric name. Avoid alerting on per-instance labels.
- **Metrics vs logs**: alerts and dashboards run on metrics. Logs are for root-causing and reports, and are usually more accurate. Even for a rare exceptional event, increment a counter and alert on the metric, not on a log search. (Workbook ch.4)

## What to instrument: "Metrics with Purpose" (Workbook ch.4)

1. **SLI metrics.** Put these on the dashboard landing page.
2. **Intended changes**: binary version, command-line flags, config version and feature-flag state. Many outages come from change, so make change visible next to the SLIs.
3. **Dependencies**: the golden signals of each direct dependency, measured from the client side.
4. **Saturation**: every resource that can run out, including thread pools, connection pools, file descriptors, quotas and queue lengths.
5. **Status of served traffic**: every response code and request class, so you can drill down.

Measure dependencies from **both sides**. Compare how fast the frontend thinks the DB is with how fast the DB thinks it is, so you can tell a DB problem from a network problem. (Book ch.6)

## Resolution and freshness

- Match the collection interval to the SLO and to how fast the metric changes. For 99.9% (about 9 hours of downtime a year), probing more than once or twice a minute is probably unnecessary. (Book ch.6)
- Get high resolution cheaply where it matters. For example, sample CPU every second into 5%-wide buckets and aggregate once a minute. A per-minute average hides spikes that drive tail latency.
- **Freshness**: data that is more than 4–5 minutes stale can significantly slow incident response. (Workbook ch.4)
- With a 99.999% SLO, a full outage uses up the monthly budget in 26 seconds, which is faster than most scrape intervals. Prevent it by design (canaries, gradual rollout), not by alerting. (Workbook ch.5)

## White-box and black-box

- Google uses **heavy white-box monitoring with modest but critical black-box monitoring**. (Book ch.6)
- **Black-box** monitoring (probers) is symptom-oriented and shows active problems as users see them. It is essential for low-traffic paths, and for failures in front of the server: DNS, load balancer, TLS.
- **White-box** monitoring (internal metrics) catches imminent problems and failures hidden by retries, and supports debugging.
- Probe both the frontends and the backends. One team's symptom is another team's cause. (Book ch.10)

## System design rules (Book ch.6, Workbook ch.4)

- **Keep it simple.** The path from onset to page to triage to debugging must be understandable by everyone on the team.
- **No "magic"** systems that learn thresholds or infer causality. The exception is simple detection of unexpected changes in end-user request rate.
- **Avoid complex dependency trees in alert logic.** Use them only for very stable rules, such as "if a datacenter is drained, don't alert on its latency".
- **Treat monitoring config as code**: version it, review it, test it and roll it out like any other change.
- **Keep the pieces loosely coupled**: collection, storage, alerting and dashboards connected by stable interfaces. Keep monitoring separate from profiling, tracing, crash reporting and log analysis.
- **Prune regularly.** Rules, signals and dashboards exercised less than once a quarter are candidates for removal, and so is anything collected that no alert or dashboard uses.
- **Give monitoring an owner.** A Google SRE team of about 10–12 people typically has one or two focused on monitoring.

## Testing monitoring (Workbook ch.4)

Alert rules may not fire for months or years, so test them in three tiers:
1. **Binary reporting**: the service exports the expected metrics with the right labels, checked by unit tests.
2. **Monitoring configuration**: the rules evaluate as intended on synthetic series. Use `promtool test rules` for Prometheus, or the equivalent for Terraform or JSON policies.
3. **Alert routing**: a synthetic alert reaches the right receiver with the right severity, and inhibition and grouping work.

Also **monitor the monitoring**: alert when the scraper or rule evaluator is down, when data is stale, or when alert delivery fails. (Book App. D)

## Dashboards

- The landing page shows SLIs and budget status, with recent changes (versions, flags, config pushes) overlaid.
- Next come the golden signals per request class, then per dependency, then saturation per resource.
- Show latency as p50/p95/p99 or as a heatmap, with successes and errors separate.
- Show burn rate with reference lines at 1×, 6× and 14.4× (for a 99.9% SLO).

## Recording rules (Prometheus)

```yaml
groups:
  - name: golden_signals
    interval: 30s
    rules:
      - record: job:http_requests:rate5m
        expr: sum by (job, code) (rate(http_requests_total[5m]))
      - record: job:http_errors:ratio_rate5m
        expr: |
          sum by (job) (rate(http_requests_total{code=~"5.."}[5m]))
          / sum by (job) (rate(http_requests_total[5m]))
      - record: job:http_latency_seconds:p99_5m          # for dashboards, not an SLI
        expr: histogram_quantile(0.99, sum by (job, le) (rate(http_request_duration_seconds_bucket{code!~"5.."}[5m])))
      - record: job:http_latency:ratio_under_300ms_5m     # SLI-style ratio
        expr: |
          sum by (job) (rate(http_request_duration_seconds_bucket{le="0.3", code!~"5.."}[5m]))
          / sum by (job) (rate(http_request_duration_seconds_count{code!~"5.."}[5m]))
      # Saturation (Kubernetes): usage against the limit, and throttling
      - record: pod:cpu_usage:ratio_limit
        expr: |
          sum by (namespace, pod) (rate(container_cpu_usage_seconds_total{container!=""}[5m]))
          / sum by (namespace, pod) (kube_pod_container_resource_limits{resource="cpu"})
      - record: pod:cpu_throttled:ratio
        expr: |
          sum by (namespace, pod) (rate(container_cpu_cfs_throttled_periods_total[5m]))
          / sum by (namespace, pod) (rate(container_cpu_cfs_periods_total[5m]))
      - record: pod:memory_working_set:ratio_limit
        expr: |
          sum by (namespace, pod) (container_memory_working_set_bytes{container!=""})
          / sum by (namespace, pod) (kube_pod_container_resource_limits{resource="memory"})
```

Saturation series feed **dashboards and tickets**. They page only when failure is definite and imminent (see `alerting.md`).

## Monitoring design checklist

- [ ] SLIs instrumented at the chosen point (load balancer, server or prober)
- [ ] Golden signals for every user-facing request class, using counters and histograms
- [ ] Latency of errors kept separate from latency of successes
- [ ] Golden signals for each dependency, measured from the client side
- [ ] Saturation of the most constrained resource, with predictive alerts only where definite
- [ ] Change events (version, flags, config) exported and overlaid on dashboards
- [ ] Bounded label cardinality, and SLO thresholds present as histogram buckets
- [ ] Monitoring config in version control, with rule tests and routing tests
- [ ] Monitoring of the monitoring pipeline itself
- [ ] A named owner and a quarterly pruning review
