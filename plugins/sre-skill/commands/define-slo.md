---
description: Define SLIs, SLOs, error budgets and burn-rate alerts for a service
argument-hint: <service, e.g. "payments API">
---

Define SLOs for: $ARGUMENTS

Use the `sre-engineer` skill and load `references/slo-sli-management.md`, `references/error-budget-policy.md` and `references/alerting-philosophy.md`.

If unclear, first ask about the service's users, its critical user journeys, and how traffic is measured today.

Produce:

1. **Critical user journeys**: what users do, and what "working" means to them.
2. **SLIs** as good events / valid events, measured from real requests. State exactly which status codes count as good and which requests are excluded (health checks, synthetic, canary traffic). For latency, use the fraction of requests faster than a threshold, not a single percentile value.
3. **SLO targets** with a user-impact justification, and the window (for example 30 days rolling).
4. **Error budget**: allowed bad events and allowed downtime per window, with the arithmetic shown.
5. **Burn-rate alerts** in Google SRE Workbook form:
   - page: 14.4× over 1h and 5m (2% of budget)
   - page: 6× over 6h and 30m (5% of budget)
   - ticket: 1× over 3d and 6h (10% of budget)
   Define every recording rule the alerts use.
6. **Error budget policy**: what changes at 50%, 25% and 0% remaining.
7. **Config** for the user's stack.
