---
description: Run a Production Readiness Review (PRR) / launch review of a service against Google SRE's launch checklist
argument-hint: <service, design doc, or repository path>
---

Run a Production Readiness Review for: $ARGUMENTS

Use the `sre-engineer` skill. Load `references/capacity-launch.md`, `references/slo-sli.md`, `references/alerting.md`, `references/release-config.md` and `references/overload-cascading.md`. Also load `references/data-state.md` if the service stores data.

If the current directory contains the service's code, infrastructure or design docs, read them first and base your findings on evidence (file paths and line numbers). Ask only for what you can't find.

Go through each row of the launch checklist in `references/capacity-launch.md`:
- architecture, N+2 redundancy and failure domains
- volume estimates 6 months out, including the launch spike
- capacity per failure domain at the latency SLO, and load tests past the breaking point
- failure handling: timeouts, retries (≤ 3 attempts, < 10% budget, backoff with jitter, one layer only), deadlines, load shedding and degradation
- monitoring and SLO burn-rate alerting, with playbooks
- release safety: canary, staged rollout, tested rollback, config validation
- dependencies: hard or soft, with fallbacks
- data integrity: tested restores and soft delete
- security basics
- client behavior
- operational load and on-call plan

Output the PRR template from `references/capacity-launch.md`:
- a findings table, with each risk ranked by its likely error budget impact
- what must be fixed before handoff or launch, and what can follow
- a decision: accept / accept with conditions / not yet

Don't change code unless the user asks.
