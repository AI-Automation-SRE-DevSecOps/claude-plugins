---
description: Define SLIs, SLOs, an error budget, burn-rate alerts and an error budget policy for a service
argument-hint: <service, e.g. "payments API">
---

Define SLOs for: $ARGUMENTS

Use the `sre-engineer` skill. Load `references/slo-sli.md`, `references/error-budget-policy.md` and `references/alerting.md`.

If it's unclear, first ask about the service's users, its critical user journeys, how traffic is measured today, and who the stakeholders are (product, dev, SRE). If the current directory has code or infrastructure for the service, read it first.

Produce:

1. **Critical user journeys**: what users do, and what "working" means to them.
2. **SLI specs and implementations**: each SLI as good events / valid events, measured as close to the user as is affordable (usually load balancer metrics). State exactly which status codes are good or bad (default: non-5xx is good; decide explicitly about 429 and client timeouts) and which requests are excluded (health checks, synthetic, canary, drained). Use no more than five SLI types. Write latency as "X% of requests faster than T", with two thresholds for the tail, never as a mean or a single percentile value.
3. **SLO targets and window**: a 4-week rolling window by default. Give the rationale: historical data rounded (availability down, latency up to 50 ms), user research, or an honest "initial guess, revisit on <date>". Say what dependencies make achievable.
4. **Error budget**: allowed bad events and allowed full-outage minutes per window, with the arithmetic shown.
5. **Burn-rate alerts**, following Workbook Table 5-8:
   - page: 14.4× over 1h AND 5m (2% of budget)
   - page: 6× over 6h AND 30m (5% of budget)
   - ticket: 1× over 3d AND 6h (10% of budget), optionally also 3× over 1d AND 2h

   Define every recording rule the alerts use. Don't use `for:` clauses. Check for low traffic and extreme SLOs (≤ 90% or ≥ 99.99%), and adjust if needed.
6. **Error budget policy** in the Workbook App. B shape:
   - If the budget for the preceding 4 weeks is exceeded, freeze changes except P0 bug fixes and security fixes until the service is back within SLO.
   - A single incident using more than 20% of the budget requires a postmortem with a P0 action item.
   - A class of outage using more than 20% in a quarter requires a P0 item in the next quarter's planning.
   - Name the escalation path and the approvers.
7. **SLO document** using the template in `references/slo-sli.md`.
8. **Config** for the user's stack.
