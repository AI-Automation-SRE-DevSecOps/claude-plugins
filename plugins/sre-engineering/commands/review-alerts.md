---
description: Review existing alert rules against Google SRE alerting and SLO burn-rate practice
argument-hint: <file, directory, or pasted alert rules>
---

Review these alerting rules: $ARGUMENTS

If no target is given, search the current directory for alert definitions, for example:
- Prometheus rule files
- Alertmanager config
- Terraform `aws_cloudwatch_metric_alarm`, `google_monitoring_alert_policy` or `datadog_monitor` resources
- Grafana alerts

Review whatever you find.

Use the `sre-engineer` skill and load `references/alerting.md` (plus `references/monitoring.md` if you need to check queries).

For each alert:

1. Answer the five review questions:
   - Is it urgent, actionable and user-visible?
   - Will it never be ignored as benign?
   - Are users definitely affected?
   - Is there an action to take?
   - Is it free of duplicate pages?
2. Check it against the four rules for pages: urgent, actionable, needs intelligence, novel.
3. Classify it as a symptom or a cause. A cause alert must be definite and imminent to page.
4. Check the query:
   - Units are correct; errors are a ratio, not a rate; percentiles are not used as SLIs.
   - Label filters exclude drained, canary, test and synthetic traffic and health checks.
   - It aggregates, rather than firing per instance.
   - Windows are right, and there are no `for:` clauses standing in for windows.
   - SLO alerts use multiwindow, multi-burn-rate with the short window at 1/12 of the long one.
   - Recording rules exist for every window referenced.
5. Decide the right output: page, ticket, dashboard only, or automate.

Then output:

- The **alert review table** from `references/alerting.md`, with a verdict per alert: keep / change / downgrade / delete / automate.
- Estimated paging load against the budget of 2 incidents per 12-hour shift.
- The proposed rule changes, as a diff or replacement config.
- **Gaps**: user-facing symptoms that no alert covers, especially missing SLO burn-rate alerts, missing inhibition, missing playbooks and missing monitoring-of-monitoring.

Don't edit files until the user agrees with the proposed changes.
