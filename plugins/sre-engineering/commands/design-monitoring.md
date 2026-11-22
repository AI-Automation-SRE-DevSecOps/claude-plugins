---
description: Design a monitoring and alerting system for a service, following Google SRE practice
argument-hint: <service or system to monitor, e.g. "checkout API on EKS">
---

Design a monitoring and alerting system for: $ARGUMENTS

Use the `sre-engineer` skill. Load `references/monitoring.md`, `references/alerting.md` and `references/slo-sli.md`, and also `references/on-call.md` if paging rotations are in scope.

If the target is unclear, first ask what the service does, who its users are, where it runs (Kubernetes, VMs, managed services; which cloud), and what monitoring stack exists. If the current directory has infrastructure code for the service, read it before designing.

Produce:

1. **Architecture**: what is measured where, and how data flows from collection to storage, alerting and dashboards. Keep the components loosely coupled, keep monitoring config in version control, and monitor the monitoring.
2. **SLIs and SLOs**: request-based ratio SLIs from real traffic (load balancer or service metrics), with targets, a 4-week window and the error budget. Use probes for low-traffic paths and for failures in front of the server.
3. **Golden signals**: for each request class and each direct dependency, cover latency (histograms with SLO thresholds as buckets, errors kept separate), traffic, error ratio, and saturation of the most constrained resource. Measure queueing or throttling, not just utilization.
4. **Metrics with purpose**: change events (version, flags, config) overlaid on SLIs, bounded label cardinality, counters rather than gauges.
5. **Granularity**: the collection interval for each metric, matched to the SLO and to how fast the metric changes. Note freshness requirements.
6. **Alerts**: multiwindow, multi-burn-rate SLO alerts. Add cause alerts only where the cause is definite and imminent. Route each alert to a page, a ticket or a dashboard, or automate it, and set inhibition for shared root causes.
7. **Alert review table**: one row per alert, as defined in `references/alerting.md`, including estimated pages per week against the budget of 2 incidents per 12-hour shift.
8. **Playbook outline** for each page.
9. **Dashboards**: a landing page with SLIs, budget and recent changes, then drill-downs.
10. **Tests**: metric export tests, rule tests (e.g. `promtool test rules`) and alert routing tests.
11. **Config as code** for the user's stack (Prometheus rules, CloudWatch alarms in Terraform, Cloud Monitoring policies, Datadog monitors and so on).
