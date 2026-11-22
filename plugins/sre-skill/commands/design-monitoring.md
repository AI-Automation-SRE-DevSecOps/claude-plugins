---
description: Design a monitoring and alerting system for a service, following Google SRE practices
argument-hint: <service or system to monitor, e.g. "checkout API on EKS">
---

Design a monitoring and alerting system for: $ARGUMENTS

Use the `sre-engineer` skill, and load `references/alerting-philosophy.md`, `references/monitoring-alerting.md` and `references/slo-sli-management.md`.

If the target is unclear, first ask what the service does, who its users are, where it runs (Kubernetes, VMs, managed services; which cloud), and what monitoring stack exists. If the current directory contains infrastructure code for it, read that before designing.

Produce:

1. **Architecture**: single-node or distributed, what is measured where, and how data flows from collection to alerts and dashboards. Keep monitoring separate from logs, traces and profiling, linked by stable interfaces.
2. **SLIs and SLOs**: request-based SLIs measured from real traffic (load balancer or service metrics), with targets and the error budget for each. Use probes only as a backup for low-traffic periods.
3. **Golden signals**: latency (percentiles, not averages), traffic, errors and saturation for each service. For saturation, name the most constrained resource and measure waiting, not only utilization (for example, CPU throttling ratio).
4. **Granularity**: the collection interval for each metric, matched to how fast it changes.
5. **Alerts**: multi-window burn-rate alerts on SLOs, plus only definite and imminent cause alerts. Route each alert to a page, a ticket or a dashboard.
6. **Alert review table**: one row per alert, as defined in `references/alerting-philosophy.md`.
7. **Runbook outline** for each page.
8. **Config**: the rules and dashboards as code for the user's stack (for example Prometheus rules, CloudWatch alarms in Terraform, or Cloud Monitoring policies).
