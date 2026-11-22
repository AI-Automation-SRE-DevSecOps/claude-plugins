# Alerting Philosophy (Google SRE)

Source: *Site Reliability Engineering* (Google, O'Reilly 2016), Chapter 6, "Monitoring Distributed Systems".

Apply this to **every alert you write or review**. A page costs human attention and sleep, so only page when a human really needs to act now.

## Three alert outputs

Every alerting rule routes to exactly one of these:

| Output | Meaning | Example |
|---|---|---|
| **Page** | A human must act **now** | Checkout error rate breaching SLO burn rate |
| **Ticket** | A human must act, but it **can wait** for business hours | TLS cert expires in 20 days |
| **Dashboard / log only** | Useful for diagnosis, not for interrupting anyone | Per-node CPU %, GC pause time |

If a condition needs a fixed, scripted response, it isn't any of these. **Automate the response** instead, and alert only if the automation fails.

## The five review questions

Answer all five for each alert. If any answer fails, downgrade the alert (page → ticket → dashboard) or redesign it.

### 1. Urgent, actionable, and user-visible?
> Does this rule detect an otherwise undetected condition that is urgent, actionable, and actively or imminently user-visible?

- All three properties are required for a page.
- "Otherwise undetected": if another alert already catches this condition, this one is redundant.

| ❌ Not a page | ✅ Page |
|---|---|
| `CPU > 80% on node-3` | `checkout 5xx ratio > 14.4 × error budget over 1h and 5m` |

### 2. Will it ever be ignored as benign?
> When and why will I be able to ignore this alert, and how can I avoid this scenario?

- An alert people learn to ignore is worse than no alert, because they'll ignore it when it's real.
- Fix: tighten the condition, use trends instead of static thresholds, or add `for:` durations.

```promql
# ❌ fires nightly while a batch job writes temp files, then cleans up
node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.10

# ✅ fires only when the disk will actually be full soon
predict_linear(node_filesystem_avail_bytes[1h], 4 * 3600) < 0
```

### 3. Are users definitely affected?
> Are there detectable cases in which users aren't being negatively impacted, such as drained traffic or test deployments, that should be filtered out?

Filter out: drained regions/clusters, canaries, load tests, synthetic traffic, non-prod environments, and health-check endpoints.

```promql
sum(rate(http_requests_total{code=~"5..", env="prod", traffic!="synthetic", handler!="/healthz"}[5m]))
```

### 4. Can someone act, and how?
> Is that action urgent, or could it wait until morning? Could the action be safely automated? Will that action be a long-term fix, or just a short-term workaround?

| Answer | Do this |
|---|---|
| No action possible | Not an alert: dashboard only |
| Action can wait | Ticket, not a page |
| Action is scripted / robotic | Automate it, and page only if automation fails |
| Action is a recurring workaround | Page for now, **and** open a ticket for the permanent fix |

### 5. Is someone else already paged for this?
> Are other people getting paged for this issue, therefore rendering at least one of the pages unnecessary?

- One root cause (e.g. the database is down) must not page every dependent team.
- Page the owner of the user-facing symptom or of the root cause, and suppress the rest with Alertmanager `inhibit_rules`, alert grouping, or dependency-aware routing.

```yaml
inhibit_rules:
  - source_matchers: [alertname="DatabaseDown"]
    target_matchers: [severity="page"]
    equal: [cluster]
```

## The four rules for pages

1. **Urgency is limited.** A human can only react urgently a few times a day. Budget roughly **≤ 2 incidents per on-call shift**; more than that means alerts need pruning.
2. **Every page is actionable.** "Look and do nothing" means it isn't a page.
3. **Every page needs intelligence.** If the runbook is "run command X and you're done", automate X.
4. **Pages are novel.** A page that fires repeatedly for a known cause signals unfixed debt. Fix the cause instead of paging again.

## Symptoms over causes

| Symptom (what users feel): **page on these** | Cause (why): **usually dashboard/ticket** |
|---|---|
| 5xx ratio breaching SLO burn rate | DB replica lag high |
| p99 latency above SLO threshold | CPU 90% on a node |
| Successful-checkout rate dropped | Pod restarted |

- Symptom alerts catch **every** cause, including ones you didn't predict. Cause alerts only catch the ones you listed.
- Page on a cause **only when it is definite and imminent**: it will certainly hurt users, soon.
  - ✅ "Disk will be full in < 4h", "TLS cert expires in < 24h", "error budget exhausted in < 2 days"
  - ❌ "CPU at 85%", "memory at 80%", "one pod restarted"
- **White-box vs. black-box doesn't matter.** If an alert passes the rules above, the data source is irrelevant.

## Required output when designing alerts

When you produce alerting rules, include an **alert review table** with one row per alert:

| Alert | Symptom or cause | Output (page/ticket/dashboard) | User impact | Action / runbook | Filters applied | Dedup / inhibition | Automatable? |
|---|---|---|---|---|---|---|---|

Flag any alert that fails a question, and say what you changed or why it stays.

## Anti-patterns to reject

- Paging on resource utilization alone (CPU %, memory %) without user impact
- Static thresholds that fire on known-benign daily patterns
- Pages without a runbook link
- Pages whose runbook is a single fixed command
- Several teams paged for one root cause
- Alerting on test, canary, drained or synthetic traffic
- `severity: critical` used for things that can wait until morning
