# Alerting and Paging

Sources: SRE Book ch.6 "Monitoring Distributed Systems", ch.10 "Practical Alerting", ch.11 "Being On-Call", ch.29 "Dealing with Interrupts". SRE Workbook ch.5 "Alerting on SLOs", ch.8 "On-Call", ch.17 "Identifying and Recovering from Overload".

Apply this to **every alert you write or review**. A page costs human attention and sleep. "Effective alerting systems have good signal and very low noise."

## Three outputs, plus "automate it"

Every rule routes to exactly one of these:

| Output | When | Example |
|---|---|---|
| **Page** | A human must act **now**; a user-visible symptom is happening or definitely imminent | Fast SLO burn: 14.4× over 1h and 5m |
| **Ticket** | A human must act, but it **can wait** for working hours | Slow burn (1× over 3d); TLS certificate expires in 20 days; failing backups for a pre-launch DB |
| **Dashboard / log** | Useful for diagnosis; nobody needs to be interrupted | Per-pod CPU %, GC pauses, a single pod restart |

If the response is fixed and scripted, it isn't any of these: **automate the response**, and alert only when the automation fails. "If a page merely merits a robotic response, it shouldn't be a page." **"Pages and tickets are the only valid ways to get a human to take action"** (Workbook ch.5). Email doesn't count.

## The five review questions (Book ch.6)

Answer all five for every alert. If any answer fails, downgrade the alert (page → ticket → dashboard) or redesign it.

1. **Does the rule detect an otherwise undetected condition that is urgent, actionable, and actively or imminently user-visible?** All three properties are required to page. If another alert already catches the condition, this one is redundant.
2. **Will I ever be able to ignore this alert as benign? When and why, and how can I avoid that?** An alert people learn to ignore is worse than none. Fix the condition: use SLO burn rates, trends or predictions instead of static thresholds.
3. **Does it definitely indicate that users are hurt?** Filter out detectable cases where they aren't: drained traffic, test deployments, canaries judged separately, load tests, synthetic traffic and health checks.
4. **Can I act on it? Is the action urgent, or could it wait until morning? Could it be safely automated? Is it a long-term fix or a short-term workaround?**
5. **Are other people being paged for this issue,** making at least one of the pages unnecessary?

## The four rules for pages (Book ch.6)

1. **Every page is urgent.** A human can react with urgency only "a few times a day" before becoming fatigued. The budget is **at most 2 incidents per 12-hour shift**, and the median should be about zero per day (Book ch.11, Workbook ch.8).
2. **Every page is actionable.** If the response is "look and do nothing", it isn't a page.
3. **Every page needs intelligence.** If the playbook is "run command X", automate X. Pages with rote, algorithmic responses "should be a red flag".
4. **Pages are about novel problems.** A page that fires repeatedly for a known cause is unfixed debt. Fix the cause, or fully automate the response.

If an alert meets these four rules, it doesn't matter whether white-box or black-box monitoring produced it. You still need both kinds of monitoring: heavy white-box, plus modest but critical black-box.

## Symptoms over causes

| Symptom (what users feel): **page on these** | Cause (why): **usually a dashboard or ticket** |
|---|---|
| Error-budget burn rate on availability | DB replica lag high |
| Latency SLO burn rate (share of slow requests) | CPU at 90% on a node |
| Successful checkout ratio dropped | One pod restarted |
| Data freshness SLO burn (pipelines) | A worker queue growing |

- Symptom alerts catch **every** cause, including ones you didn't predict. "It's better to spend much more effort on catching symptoms than causes."
- Page on a cause **only when it is very definite and very imminent**. Running at N+0 redundancy and "nearly full" resources count as imminent (Book ch.6 fn.25):
  - ✅ "disk full in < 4 h" (predicted), "TLS cert expires in < 24 h", "cluster down to N+0 at peak", "quorum about to be lost"
  - ❌ "CPU at 85%", "memory at 80%", "one pod restarted", "one replica down while N+2 still holds"
- One team's cause is another team's symptom. Slow DB reads are a symptom for the DB team and a cause for the frontend team.
- **Never page on single machines or tasks.** Their data is "too noisy to be actionable"; alert on aggregates (Book ch.10).

## SLO burn-rate alerting (Workbook ch.5)

Evaluate any SLO alerting strategy on four properties: **precision**, **recall**, **detection time** and **reset time**.

### The six approaches, and why the last one wins

| # | Approach | Verdict |
|---|---|---|
| 1 | Error rate ≥ SLO over a short window (e.g. 10m) | ❌ Low precision: up to 144 alerts/day while still meeting the SLO |
| 2 | Error rate ≥ SLO over a long window (e.g. 36h) | ❌ Reset time is terrible: fires for 36h after a short outage. Expensive to compute |
| 3 | Short window plus `for: 1h` | ❌ **Not recommended.** A 100% outage pages after an hour, by which time it has burned 140% of the budget. Periodic spikes never alert |
| 4 | Single burn rate (e.g. 36× over 1h) | ⚠️ Low recall: a 35× burn never alerts but uses the whole budget in about 20.5h |
| 5 | Multiple burn rates | ⚠️ Good recall and precision, but long reset times and duplicate notifications without suppression |
| 6 | **Multiwindow, multi-burn-rate** | ✅ **Recommended.** Each long window is ANDed with a short window |

### Recommended parameters for a 99.9% SLO (Workbook Table 5-8)

| Severity | Long window | Short window | Burn rate | Budget consumed when it fires |
|---|---|---|---|---|
| **Page** | 1 hour | 5 minutes | 14.4 | 2% |
| **Page** | 6 hours | 30 minutes | 6 | 5% |
| **Ticket** | 3 days | 6 hours | 1 | 10% |

- The **short window is 1/12 of the long window.** It makes the alert reset within minutes once the problem ends.
- The Workbook's example config adds a ticket condition at **3× over 24h AND 2h**. It's optional.
- Threshold = `burn_rate × (1 − SLO)`. For 99.9%: 0.0144, 0.006, 0.001 (0.003 for the optional ticket).
- Budget consumed = `burn_rate × long_window / SLO_window`. For example, 14.4 × 1h / 720h = 2%. These percentages assume a 30-day window. On a 28-day window, the same burn rates consume slightly more budget per alert, about 2.1%, 5.4% and 10.7%. Keep the burn rates and note the difference.
- **Busier services, or teams that worry about weekends and holidays, may make the 6h/30m alert a ticket** instead of a page. That is a team decision.
- **Don't add `for:` clauses.** "We do not recommend using durations as part of your SLO-based alerting criteria." The short window already filters out momentary blips.
- **Use the same parameters for every service.** "We strongly advise against specifying the alert window and burn rate parameters independently for each service." Group request types into buckets (CRITICAL, HIGH_FAST, HIGH_SLOW, LOW, NO_SLO) instead.

### Edge cases

- **Low traffic**: at 10 requests/hour, one failure is a 1,000× burn on 99.9% and uses 13.9% of a 30-day budget. Options: synthetic traffic, combining services that share a failure domain, client retries with backoff, or a lower SLO or longer window. The last one is a product decision.
- **Very low SLOs (≤ 90%)**: a 100% outage burns only 1.4% per hour, so the 2%-in-1h page can never fire. Re-derive the parameters.
- **Very high SLOs (≥ 99.99%)**: at 99.999%, a full outage exhausts the budget in 26 seconds, which alerting can't defend. Rely on gradual rollout. Canarying to 1% of users first gives 43 minutes.

### Latency SLO alerts

Use the same burn-rate structure, with "bad" meaning **requests slower than the threshold**:

```promql
# 1h ratio of slow requests (>300 ms) for a "99% under 300 ms" SLO. Budget = 0.01.
1 - (
  sum by (job) (rate(http_request_duration_seconds_bucket{le="0.3"}[1h]))
  / sum by (job) (rate(http_request_duration_seconds_count[1h]))
)  > (14.4 * 0.01)
```

Don't page on `histogram_quantile(0.99, …) > 0.5` with `for: 5m`. That is approach 3 applied to a percentile.

## Silences, inhibition and deduplication

- **One root cause shouldn't page several teams.** Page the owner of the user-facing symptom or of the root cause, and inhibit dependent alerts.

```yaml
# Alertmanager: while a cluster-wide DB outage pages, suppress dependent service pages in that cluster
inhibit_rules:
  - source_matchers: [alertname="DatabaseUnavailable", severity="page"]
    target_matchers: [severity="page"]
    equal: [cluster]
route:
  group_by: [alertname, job, cluster]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
```

- **Silences must expire.** Keep them to "typically a day, sometimes up to a week", and link each one to a bug, so a silence can't hide the next outage (Workbook ch.17).
- When a page can't be fixed yet, silence it until its fix is due rather than letting it page repeatedly (Book ch.29).

## Managing alerts over time (Workbook ch.8)

- **New alerts:** run them in test mode first (non-paging, for example to the author or a ticket queue) for **about a week**. Estimate how much of the pager budget they will use, and have the team explicitly approve or reject them.
- **Relaxing thresholds is "rarely an appropriate response"** to being paged. Fix the cause, or move the alert to SLO burn.
- **Never close a page as "transient", "fixed itself" or "cause unknown".** Find the root cause, or add the monitoring or logging you would have needed.
- Link every page to a bug, rank bugs by how many pages they caused, and review pager load at the weekly production meeting using a **21-day trailing average**.
- When reviewing non-paging events (Book ch.31), sort them three ways: should have paged (fix the monitoring), needs attention but not a page (keep it as a ticket), or needs no attention (delete it).
- Every page needs a **playbook entry**, with the minimum structured content your team agrees on. If the playbook is a fixed list of commands, automate it.

## Required output: alert review table

When producing or reviewing alerting rules, include one row per alert:

| Alert | Symptom or cause | Output (page/ticket/dashboard/automate) | User impact | Action / playbook | Filters applied | Dedup / inhibition | Est. pages per week | Verdict (keep/change/downgrade/delete/automate) |
|---|---|---|---|---|---|---|---|---|

Flag every alert that fails a review question, and say what you changed or why it stays.

## Anti-patterns to reject

- Paging on resource utilization alone (CPU %, memory %) with no user impact
- Paging per instance or per pod, or on single-machine data
- Short-window error rate ≥ SLO, or `for:` durations used instead of windows
- Single-window burn rates (they miss slow burns), or burn rates tuned separately per service
- Alerting on error **rate** (errors per second) instead of error **ratio**
- Static thresholds that fire on known-benign daily patterns, such as batch jobs or nightly cleanup
- Pages without a playbook, or pages whose playbook is one fixed command
- Several teams paged for one root cause
- Alerting on test, canary, drained or synthetic traffic, or on health-check endpoints
- `severity: critical` on things that can wait until morning
- Email alerts, and silences with no expiry
- "Magic" anomaly-detection pages with no clear failure definition
