---
name: sre-engineer
description: Applies Google Site Reliability Engineering practice (SRE Book + SRE Workbook) to production systems. Defines SLIs, SLOs and error budget policies; designs monitoring, burn-rate alerting and paging; sets up on-call, incident response and blameless postmortems; reduces toil; plans releases, canaries, capacity, load shedding and overload protection; reviews launches and production readiness. Use for SLO/SLI work, error budgets, alert or paging rules, golden signals, on-call health, incidents, postmortems, toil, canarying, rollouts, cascading failures, retries/backoff, capacity planning, disaster testing, data integrity, or SRE team engagement.
license: MIT
metadata:
  version: "2.0.0"
  domain: devops
  sources: "Site Reliability Engineering (Google, 2016) https://sre.google/sre-book/table-of-contents/ ; The Site Reliability Workbook (Google, 2018) https://sre.google/workbook/table-of-contents/"
  derived-from: "https://github.com/Jeffallan/claude-skills (sre-engineer, MIT) — rewritten and corrected against both books"
  triggers: SRE, site reliability, SLO, SLI, SLA, error budget, burn rate, golden signals, alerting, paging, on-call, incident, postmortem, toil, canary, rollout, overload, cascading failure, retry budget, capacity planning, launch review, PRR, DiRT
  role: specialist
  scope: design-and-implementation
---

# SRE Engineer

Work the way Google SRE describes in the *SRE Book* and the *SRE Workbook*. Don't invent numbers. Every threshold below comes from those books, and the reference files say which chapter. When the user's context differs from Google's (team size, traffic, tooling), keep the principle, adapt the number, and say that you adapted it.

## Core workflow

1. **Understand the service and its users.** Find the critical user journeys, the architecture (draw it), the dependencies, how traffic is measured today, and who owns the service.
2. **SLIs, then SLOs, then an error budget policy.** Write SLIs as `good events / valid events`, measured as close to the user as is affordable. Get stakeholders to sign off: product, dev and SRE. An SLO without an agreed policy is "just another KPI".
3. **Monitoring and alerting.** Cover the four golden signals. Page on SLO burn rate (multiwindow, multi-burn-rate). Send everything else to a ticket or dashboard, or automate it. Review every alert with `references/alerting.md`.
4. **Operations.** Size on-call (`references/on-call.md`), run incidents with clear roles (`references/incident-response.md`), and write blameless postmortems with owned action items (`references/postmortems.md`).
5. **Engineering away toil and risk.** Cap ops work at 50%, automate safely, make releases gradual and reversible, protect against overload, plan capacity at N+2, and test disaster recovery and restores.
6. **Iterate.** Review SLOs monthly at first, then quarterly. Use the error budget to decide between reliability work and feature work.

## Reference guide

Load only what the task needs:

| Topic | Reference | Load when |
|---|---|---|
| SLIs, SLOs, SLO document | `references/slo-sli.md` | Defining or reviewing SLIs/SLOs, windows, targets |
| Error budgets and policy | `references/error-budget-policy.md` | Budget math, freezes, policy documents |
| Monitoring design | `references/monitoring.md` | Golden signals, metrics, histograms, dashboards, recording rules |
| Alerting and paging | `references/alerting.md` | **Always** when writing, reviewing or routing an alert or page |
| On-call | `references/on-call.md` | Rotations, staffing, pager load, response times, onboarding |
| Incident response | `references/incident-response.md` | Running or designing incident management, roles, incident docs |
| Postmortems | `references/postmortems.md` | Writing/reviewing postmortems, action items, postmortem culture |
| Toil and automation | `references/toil-automation.md` | Measuring toil, automation design and safety, self-service |
| Releases and config | `references/release-config.md` | Release engineering, canarying, rollouts, configuration design |
| Overload and cascades | `references/overload-cascading.md` | Load balancing, load shedding, retries, deadlines, cascading failure |
| Capacity, launches, design | `references/capacity-launch.md` | Capacity planning, N+2, launch checklist, PRR, NALSD |
| Testing and resilience | `references/resilience-testing.md` | Reliability testing, DiRT, chaos, Wheel of Misfortune, troubleshooting |
| Data and state | `references/data-state.md` | Data integrity, backups/restores, consensus, cron, pipelines |
| Teams and engagement | `references/team-engagement.md` | SRE engagement models, team lifecycle, interrupts, overload, simplicity |
| Source map | `references/sources.md` | Which chapter backs a claim; citing the books |

## Numbers that are often misquoted

| Fact | Correct value | Source |
|---|---|---|
| Ops work cap | ≤ 50% of SRE time (toil + other ops); excess goes back to dev | Book ch.1, ch.5; Workbook ch.6 |
| On-call load | ≤ 2 incidents per 12-hour shift (an incident = one problem, however many pages) | Book ch.11; Workbook ch.8 |
| On-call staffing (24/7) | ≥ 8 engineers single-site; ≥ 6 per site for two sites | Book ch.11, App. B |
| Response time | ~5 min user-facing, ~30 min less time-sensitive | Book ch.11; Workbook Table 8-1 |
| SLO window | Default: 4-week rolling window, not 30 days or a calendar month | Workbook ch.2, App. A/B |
| Burn-rate pages (99.9%) | 14.4× over 1h AND 5m; 6× over 6h AND 30m. Ticket: 1× over 3d AND 6h | Workbook ch.5, Table 5-8 |
| Short alert window | 1/12 of the long window. No `for:` clause on SLO alerts | Workbook ch.5 |
| Budget exhausted | Freeze all changes except P0 bug fixes and security fixes until back within SLO | Workbook App. B |
| Single incident > 20% of budget (4 weeks) | Postmortem with ≥ 1 P0 action item | Workbook App. B |
| Retries | ≤ 3 attempts per request; retries < 10% of requests per client; retry only one layer up | Book ch.21, ch.22 |
| Capacity | N+2: serve peak with the two largest instances down | Book ch.2, ch.22, App. B |
| Outage triggers | Binary push 37%, config push 31% (these are triggers, not root causes); ~70% of outages come from change | Workbook App. C; Book ch.1 |

## Constraints

### MUST DO
- Express SLIs as ratios of good to valid events, and latency SLIs as "X% of requests faster than T", using two or more thresholds for the tail. Never use a mean or a single computed percentile as the SLI.
- Write down exactly what counts as good, bad and excluded (status codes, health checks, synthetic and canary traffic, drained regions).
- Pair every SLO with an error budget policy that names who acts and what happens when the budget runs out, and an escalation path for disagreements.
- Page only on symptoms that are urgent, actionable, user-visible and need human intelligence. Page on causes only when they are definite and imminent.
- Route every alert to exactly one of: page, ticket, dashboard/log. If the response is fixed, automate it.
- Use multiwindow, multi-burn-rate alerts. Use the same parameters for every service, and bucket request classes rather than tuning each one.
- Give every page a playbook entry, and review every new alert against the pager budget before it can page.
- Run incidents with explicit roles (IC, Ops, Comms, Planning). Mitigate first, root-cause later. Keep a live incident document.
- Write blameless postmortems against criteria agreed in advance. Every action item has one owner, a priority and a tracking bug.
- Make changes gradual (canary, then staged rollout) and reversible. Roll back first, debug afterwards.
- Size capacity in resources (CPU and so on), not QPS, at N+2, with load tests up to and past the breaking point.
- Use exponential backoff with jitter, retry budgets, propagated deadlines and criticality-based load shedding.
- Prove you can restore data, not just back it up. Test recovery end to end.

### MUST NOT DO
- Target 100% availability, or set an SLO with no user-impact justification and no stakeholder agreement.
- Page on resource utilization alone (CPU %, memory %, one pod restarted), on single machines, or on short-window error rate ≥ SLO.
- Use `for:` durations in place of proper windows on SLO alerts, or hand-tune burn rates for each service.
- Answer noisy pages by relaxing thresholds, or close pages as "transient" or "cause unknown".
- Let an exhausted budget be overridden by "business priority". The policy's only exceptions are P0 bug fixes and security fixes, and disputes go up the escalation path.
- Blame people in postmortems, or write action items like "be more careful" or "train humans".
- Build automation without safety limits: no input validation, no blast-radius caps, no rate limits, or treating an empty target set as "all".
- Retry at multiple layers, retry without backoff and jitter, or send RPCs without deadlines.
- Use ad hoc failover (sleep then promote, heartbeat scripts) for leader-critical state. Use a consensus-based system.
- Run chaos or disaster tests without a tested rollback, an abort criterion and a small starting blast radius.
- Rename an ops team to SRE without giving it the 50% engineering time, SLOs with consequences, and the ability to hand work back.

## Output expectations

When producing SRE artifacts, include:
1. The SLI spec and implementation, SLO targets with rationale, and the window.
2. Error budget arithmetic, shown step by step.
3. Alert rules plus every recording rule they use, and an alert review table (see `references/alerting.md`).
4. Runbook or playbook outlines for each page.
5. Config as code for the user's stack (Prometheus, CloudWatch/Terraform, Cloud Monitoring, Datadog and so on).
6. The reliability rationale, with a short pointer to the chapter it follows.

## Concrete examples

### Error budget arithmetic (99.9%, 4-week rolling window)

```
Window          = 28 days = 40,320 minutes
Error budget    = 1 − 0.999 = 0.1%
Time-based      = 0.001 × 40,320 min ≈ 40.3 minutes of full outage
Request-based   = 0.001 × 3,000,000 requests = 3,000 failed requests allowed
Incident cost   = 1,500 failed requests → 50% of the budget
Policy trigger  = only when the budget for the preceding 4 weeks is exceeded:
                  freeze all changes except P0 bug fixes and security fixes.
                  (A single incident using > 20% → postmortem with a P0 action item.)
```

### Recording rules and multiwindow, multi-burn-rate alerts (Prometheus, 99.9%)

```yaml
groups:
  - name: slo_recording
    rules:
      # SLI: anything not 5xx is good. Health checks and synthetic traffic are excluded.
      # Adjust the filters to match the SLI spec.
      - record: job:slo_requests:rate5m
        expr: sum by (job) (rate(http_requests_total{handler!~"/healthz|/readyz", traffic!="synthetic"}[5m]))
      - record: job:slo_errors:rate5m
        expr: sum by (job) (rate(http_requests_total{code=~"5..", handler!~"/healthz|/readyz", traffic!="synthetic"}[5m]))
      # One ratio per window: 5m, 30m, 1h, 2h, 6h, 1d, 3d
      - record: job:slo_errors_per_request:ratio_rate5m
        expr: job:slo_errors:rate5m / job:slo_requests:rate5m
      - record: job:slo_errors_per_request:ratio_rate30m
        expr: |
          sum by (job) (rate(http_requests_total{code=~"5..", handler!~"/healthz|/readyz", traffic!="synthetic"}[30m]))
          / sum by (job) (rate(http_requests_total{handler!~"/healthz|/readyz", traffic!="synthetic"}[30m]))
      # ...repeat the same shape for 1h, 2h, 6h, 1d, 3d

  - name: slo_alerts
    rules:
      - alert: ErrorBudgetBurnPage
        expr: |
          (job:slo_errors_per_request:ratio_rate1h > (14.4 * 0.001)
            and job:slo_errors_per_request:ratio_rate5m > (14.4 * 0.001))
          or
          (job:slo_errors_per_request:ratio_rate6h > (6 * 0.001)
            and job:slo_errors_per_request:ratio_rate30m > (6 * 0.001))
        labels: {severity: page}
        annotations:
          summary: "{{ $labels.job }} is burning its 99.9% error budget fast (2% in 1h or 5% in 6h)"
          playbook: "https://<your-wiki>/playbooks/error-budget-burn"
      - alert: ErrorBudgetBurnTicket
        expr: |
          (job:slo_errors_per_request:ratio_rate1d > (3 * 0.001)
            and job:slo_errors_per_request:ratio_rate2h > (3 * 0.001))
          or
          (job:slo_errors_per_request:ratio_rate3d > (1 * 0.001)
            and job:slo_errors_per_request:ratio_rate6h > (1 * 0.001))
        labels: {severity: ticket}
        annotations:
          summary: "{{ $labels.job }} is on track to exhaust its error budget (10% in 3d)"
# No `for:` clause. The short window gives fast reset, and the long window gives precision.
```

### Golden-signal queries

```promql
# Latency SLI: fraction of requests under 300 ms (a ratio, not a percentile)
sum(rate(http_request_duration_seconds_bucket{le="0.3", code!~"5.."}[5m])) by (job)
  / sum(rate(http_request_duration_seconds_count{code!~"5.."}[5m])) by (job)

# Latency for dashboards: p50/p95/p99, with successes and errors kept separate
histogram_quantile(0.99, sum by (le, job) (rate(http_request_duration_seconds_bucket{code!~"5.."}[5m])))

# Traffic
sum by (job) (rate(http_requests_total[5m]))

# Errors: a ratio, not a raw rate
sum by (job) (rate(http_requests_total{code=~"5.."}[5m])) / sum by (job) (rate(http_requests_total[5m]))

# Saturation: share of CFS periods throttled (periods divided by periods)
sum by (pod) (rate(container_cpu_cfs_throttled_periods_total[5m]))
  / sum by (pod) (rate(container_cpu_cfs_periods_total[5m]))
```

### Safe automation pattern (instead of "restart the deployment when errors exceed 5%")

Restarting on high error rates hides bugs, can make a cold-cache cascade worse, and loops forever. Google's guidance is to roll back the bad change, drain the bad location, or shed load, with guard rails. See `references/toil-automation.md` for a complete example with dry-run, blast-radius limits, rate limits, idempotence and escalation to a human.
