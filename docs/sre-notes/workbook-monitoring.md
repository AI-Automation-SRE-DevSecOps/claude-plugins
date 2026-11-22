# The Site Reliability Workbook, Chapter 4: Monitoring

Source: https://sre.google/workbook/monitoring/
Authors: Jess Frame, Anthony Lenton, Steven Thurgood, Anton Tolchanov, Nejc Trdin, with Carmela Quinito.
Scope: The chapter focuses on **metrics and structured logging**. It deliberately leaves out distributed tracing and runtime/event introspection.

## Core principles

- **Why monitor** (from SRE book Ch. 6): to alert, to diagnose, to display, to find trends for planning, and to compare before and after a change or across experiment groups. Their relative importance drives the tradeoffs when you pick a system.
- **Metrics vs. logs** (*Sources of Monitoring Data*). Metrics are "numerical measurements representing attributes and events" sampled at regular intervals. They are less granular but near real time. Logs are "an append-only record of events". They are highly granular, arrive with some delay, and are "nearly always" more accurate. Google uses metrics for alerts and dashboards, and logs to find root causes and to produce detailed reports that aren't time-sensitive.
- **Treat the monitoring system as a production service** (*Managing Your Monitoring System*).
- **Every metric needs a purpose** (*Implementing Purposeful Metrics*). The purpose can be capacity planning, debugging, or direct notification of problems.
- **Alerting metrics and debugging metrics have different requirements.** An alerting metric should change dramatically only when the system enters a problem state. A debugging metric points at a possible cause.

## Concrete numbers, thresholds and formulas

- **Freshness:** "Data more than four to five minutes stale might significantly impact how quickly you can respond to an incident." (*Speed*)
- **Retention:** keep data "over a multimonth time frame" at minimum. Summary (aggregated) data is enough for growth planning. Detailed per-metric data helps with questions like "has this happened before?" but costs more. (*Calculations*)
- **Counters:** metrics should ideally be "monotonically incrementing counters" so the system can compute windowed rates. Computing rates over windows "up to a month" gives you the building blocks for SLO burn-based alerting (Ch. 5).
- **Percentiles:** the 50th, 95th and 99th percentiles show whether "50%, 5%, or 1%" of requests are too slow. A mean can only tell you that requests got slower. If the system can't compute percentiles, the chapter gives two fallbacks:
  - a mean: total seconds spent in requests ÷ number of requests;
  - logging every request and computing percentiles by scanning or sampling the logs.
- **Severity example** (*Alerts*): a low error rate that lasts **more than an hour** gets a ticket. A **100% error rate** is an emergency that needs an immediate response.
- **Label cardinality:** HTTP status code works as a label (`requests_total{status=404}` vs `{status=500}`) because it has few values. An entity ID that "could take on millions of different values" does not.

## Recommended practices and procedures

**Desirable features of a monitoring system**
- *Speed:* work out your requirements for data freshness and retrieval speed before you choose. To speed up slow graphs, have the system create and store new time series from incoming data so common queries are precomputed.
- *Interfaces:* support graphs, tables, heatmaps, histograms and log-scale graphs. Build dashboards for each audience, keep them consistent, and practice ad hoc drill-downs (machine type, server version, request type).
- *Alerts:* use alert categories and severity levels so the response can be proportional. Use suppression: alert once on the global error rate rather than once per node, and don't alert on your own error rates when a dependency's alert is already firing. Make sure suppression ends when the event is over.

**Logs vs. metrics**
- Even for "notify on a single exceptional event", use **metrics-based alerting**: increment a counter when the event happens and alert on it. This keeps all alert configuration in one place.
- *Examples:* App Engine moved status codes into metric labels, which allowed separate client and server error thresholds. Ads built a shared library that classifies user impact at request time and writes it to logs and metrics together, which cut false positives "significantly". For high-cardinality data, keep logs as the source and put the query script in the alert email.

**Managing the system**
- Treat monitoring configuration as code: revision control, linting (e.g., `promtool`), code review, rollbacks. Prefer intent-based configuration over web UIs or CRUD-style APIs. `grafanalib` is an example tool.
- Encourage consistency. Google converged on one centrally run framework, with discoverable dashboards. Make basic coverage effortless: if every service exports common metrics (via OpenCensus or Istio), new components get monitoring automatically.
- Prefer loose coupling. Use separate components with stable interfaces for collection, storage, alerting and visualization. Examples: Prometheus server, InfluxDB, Alertmanager and Grafana (vs. the all-in-one Zabbix). Instrumentation standards: statsd and Prometheus/OpenMetrics. In the Borgmon-to-Monarch move, Google split dashboarding into Viceroy, which read from both, so users could migrate gradually.

**Metrics with Purpose (checklist)**
- **SLI metrics** go first and most prominently, ideally on the dashboard's landing page.
- **Intended changes:** monitor the binary version, command-line flags (especially feature flags), and the version of any dynamically pushed configuration. Where something isn't versioned, monitor the timestamp of its last build or package. Link a change graph from the alert.
- **Dependencies:** for each direct dependency, export request and response size in bytes, latency, and response codes, using the four golden signals as a guide. Add labels for response code, RPC method and peer job. Instrument the lower-level RPC client library once rather than each client. For opaque single-RPC APIs (`Get`, `Query`), either export tailored metrics that unpack the request, or ask the owners to split the API.
- **Saturation:** track every resource, both those with hard limits (RAM, disk, CPU quota) and those without (file descriptors, thread-pool threads, queue wait times, log volume). Add language runtime resources: Java heap, metaspace and GC-specific metrics, and Go goroutine counts. Alert when a resource approaching exhaustion has a hard limit, or when crossing a usage threshold degrades performance.
- **Status of served traffic:** monitor **all** HTTP response codes, even ones too weak to alert on. Monitor aggregate counts of requests denied for lack of quota or by rate limits.
- **In postmortems,** ask which additional metrics would have let you diagnose the issue faster.

## Template: three-tier monitoring testing (Figure 4-1)

1. **Binary reporting:** exported metric variables change value as expected under certain conditions.
2. **Monitoring configurations:** rule evaluation gives the expected results, and specific conditions produce the expected alerts.
3. **Alerting configurations:** generated alerts are routed to a predetermined destination based on their label values.

Google uses a DSL to create synthetic time series and asserts on derived series and on alert firing status and labels. Where a stage can't be tested synthetically, run a live system that exports well-known metrics (requests, errors) to validate against.

## Anti-patterns

- Acting on stale data, which can suggest a change had no effect or a false correlation.
- Using the mean for latency.
- A separate logs-based alerting path.
- High-cardinality metric labels (entity IDs).
- Per-service log scripts as the SLO source of truth (the Ads case, about 50 services).
- UI-only monitoring configuration.
- Dashboards bundled with alert rules (Borgmon).
- Exporting metrics "just because they are easy to generate."
- Untested alert rules.

## Short verbatim quotes

- "Your monitoring system is as important as any other service you run."
- "Each exposed metric should serve a purpose."
- "Logs will nearly always produce more accurate data than metrics."
- "Ideally, metric values used for alerting change dramatically only when the system enters a problem state."
- "It's very likely that your alerting rules will not fire for months or years after you configure them."
