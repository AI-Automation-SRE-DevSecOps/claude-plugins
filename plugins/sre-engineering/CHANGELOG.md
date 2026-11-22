# Changelog

## 2.0.0: renamed `sre-skill` to `sre-engineering`, rewritten against the SRE Book and SRE Workbook

Every chapter and appendix of both books was read and the skill was rebuilt from those notes. `references/sources.md` maps each chapter to the reference file that covers it.

### Breaking changes
- Plugin renamed from `sre-skill` to `sre-engineering`. Commands are now `/sre-engineering:<command>`. The skill keeps the name `sre-engineer`.
- Reference files reorganized:

| Old file | New file(s) |
|---|---|
| `slo-sli-management.md` | `slo-sli.md` |
| `error-budget-policy.md` | `error-budget-policy.md` (rewritten) |
| `monitoring-alerting.md` | `monitoring.md` + `alerting.md` |
| `alerting-philosophy.md` | `alerting.md` |
| `automation-toil.md` | `toil-automation.md` + `capacity-launch.md` |
| `incident-chaos.md` | `incident-response.md` + `postmortems.md` + `resilience-testing.md` |
| — | `on-call.md`, `release-config.md`, `overload-cascading.md`, `data-state.md`, `team-engagement.md`, `sources.md` (new) |

- New commands: `write-postmortem` and `production-readiness-review`.

### Corrections (what was wrong, according to Google's books)

**Burn-rate alerting (Workbook ch.5)**
1. The slow-burn page used **1× over 6h** (threshold `0.001`). It is now **6× over 6h AND 30m** (`0.006`).
2. The medium alert used **6× over 6h AND 1d**. The short window must be **1/12 of the long window**, so it is now 6h AND 30m.
3. `for: 2m` / `for: 15m` were added to SLO alerts. The Workbook says not to use durations in SLO alerting criteria. They are removed.
4. The alerts referenced `error_rate1h`, `6h` and `1d` recording rules that were never defined. All of them are now defined.
5. The Python burn-rate checker tested single windows only. Multiwindow logic and the ticket tier are now documented.
6. The latency alert was `p99 > 500ms for 5m`, which is approach 3 applied to a percentile. It is now a burn rate on the share of slow requests.

**SLIs and SLOs (Workbook ch.2, App. A; Book ch.4, App. A)**

7. The latency SLI compared a **single p95 value** with 500 ms, under a 99% "ratio" target. It is now a ratio of requests faster than the threshold, using histogram buckets.
8. Good events were defined inconsistently: some queries counted 2xx+4xx, others only 2xx. It is now one definition (non-5xx is good), with explicit guidance on 429 and client timeouts.
9. The default window was **30 days**. The Workbook recommends a **4-week rolling window**, and the arithmetic is updated.
10. The tier table had inconsistent downtime figures (4m23s, 43m28s and 3h37m don't come from one month length) and presented arbitrary latency targets as standards. It is replaced with the Book's availability table and the Workbook's illustrative request-class buckets.
11. The golden-signals `is_healthy()` treated CPU and memory as SLO targets. Saturation is a diagnostic signal, not an SLO.
12. Missing material added: SLI menu, spec vs implementation, prerequisites for an SLO, rounding rules for starter SLOs, the Table 2-5 decision matrix, dependency math, low-traffic handling, and an SLO document template.

**Error budget policy (Workbook App. B)**

13. The invented **50%/25% tiers with "senior/VP approval for deploys"** are removed. Google's policy freezes changes **only once the budget for the preceding four weeks is exceeded**, and exempts only **P0 bug fixes and security fixes**.
14. "Critical business requirement" was an exemption, and `should_deploy()` let `business_priority == "critical"` override an exhausted budget. Disagreements now go up the **escalation path** (CTO) instead.
15. Missing rules added: a **single incident using more than 20%** of the budget requires a postmortem with a P0 action item, and an **outage class using more than 20% in a quarter** requires a P0 item in planning. Data changes are frozen as well as binaries.
16. The SKILL.md example froze releases after 50% of the budget was used in week 1. It now uses the real policy trigger.

**Monitoring and paging (Book ch.6, ch.10, ch.11; Workbook ch.4, ch.8)**

17. `HighMemoryUtilization > 85%` was a warning-level cause alert, which contradicted the skill's own philosophy. Saturation now goes to dashboards and tickets, and pages only when failure is definite and imminent.
18. The CPU saturation query divided `throttled_seconds_total` by `cfs_periods_total`, which mixes units. It now uses `throttled_periods_total / cfs_periods_total`.
19. "Fix benign alerts by adding `for:` durations" is replaced with SLO burn rates, trends and predictions.
20. On-call guidance like "Predictive alerts are never page-worthy", ">5% users affected" and ">10× burn" is replaced with Google's rules: definite and imminent causes may page, and the 14.4×/6× thresholds apply.
21. `AlertQualityMetrics.toil_ratio` counted actionable alerts as toil, which made no sense. It is replaced with pager-load tracking: a 21-day trailing average against 2 incidents per 12-hour shift.
22. Added: page/ticket/log outputs (email isn't one), the four rules for pages, never paging on single machines, new-alert trials, "relaxing thresholds is rarely appropriate", silences that expire, and monitoring tests in three tiers.

**Toil and automation (Book ch.5, ch.7, ch.13; Workbook ch.6)**

23. The toil definition was missing **tactical**, **no enduring value** and **O(n)**, and the toil attributes were modelled as mutually exclusive "categories". It now has all six attributes, treated as a spectrum, with overhead distinguished from toil.
24. The 50% cap was presented as a weekly toil metric. It is now a cap on **all operational work**, averaged over quarters.
25. The auto-remediation scripts restarted deployments when the error rate passed 5%, and deleted logs. That hides bugs, can amplify cold-cache cascades and loops forever. They are replaced with a guarded example: dry-run, blast-radius limits, cooldown, idempotence, an empty-set check ("zero does mean all") and escalation.
26. The database failover runbook (`sleep 10`, promote, repoint the selector to `role=replica`) was unsafe: no fencing, split-brain risk, and the selector was wrong. It is replaced with guidance to use consensus-based failover (Book ch.23).
27. Capacity planning used a linear fit of CPU % against 80%. It now uses organic plus inorganic demand forecasts, load-tested capacity per unit at the latency SLO, and **N+2** per failure domain.

**Incidents, postmortems and chaos (Book ch.12–15, ch.17, ch.33; Workbook ch.9, ch.10)**

28. The incident roles lacked an **Ops lead** and a **Planning lead**, and gave the on-call engineer "investigate root cause" during the incident. They now follow the ICS-based roles: IC, Ops, Comms, Planning. Mitigate first, and only Ops touches production.
29. Added Google's incident declaration criteria, explicit handoffs, the live incident document, and the generic mitigations.
30. The postmortem template had no trigger/root-cause split, no action-item **type** and no bug links, and was written for "all incidents". It now has agreed triggers, typed action items with one owner each, and the >20% budget rule.
31. The chaos runner's `business_hours_only` check was **inverted**: it aborted *during* business hours. Google says to test while your best people are present, so the check is fixed.
32. The example chaos experiment killed **50% of production pods** with a 10% error-rate abort. Experiments now start with the smallest blast radius, require a tested rollback, gate on error budget, and abort on SLO burn rate.
33. A game-day expectation of "10× traffic → no errors" was unrealistic. Overload handling expects graceful degradation and load shedding.

### Added coverage
On-call staffing and load; release engineering and canarying; configuration design; load balancing, retries, deadlines, criticality and cascading failures; NALSD; launch coordination and PRR; DiRT, Wheel of Misfortune and restore testing; data integrity, consensus, distributed cron and pipelines; interrupts, operational overload, simplicity, engagement models and team lifecycle.

## 1.0.0
Initial import of `Jeffallan/claude-skills` `sre-engineer` (commit `4b94d98`), with an added alerting philosophy and three commands.
