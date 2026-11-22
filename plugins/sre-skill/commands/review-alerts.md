---
description: Review existing alert rules against Google SRE alerting philosophy
argument-hint: <file, directory, or pasted alert rules>
---

Review these alerting rules: $ARGUMENTS

If no target is given, search the current directory for alert definitions (for example Prometheus rule files, Alertmanager config, Terraform `aws_cloudwatch_metric_alarm` or `google_monitoring_alert_policy` resources, Grafana alerts) and review those.

Use the `sre-engineer` skill and load `references/alerting-philosophy.md`.

For each alert:

1. Answer the five review questions: urgent/actionable/user-visible, never benign, users definitely affected, action exists, no duplicate pages.
2. Classify it as a symptom or a cause. A cause alert must be definite and imminent to page.
3. Check the query is correct: units, label filters (drained, canary, test, synthetic traffic), time windows, and `for:` durations.
4. Decide the right output: page, ticket, dashboard only, or automate.

Then output:

- The **alert review table** from `references/alerting-philosophy.md`, with a verdict per alert (keep / change / downgrade / delete / automate).
- The proposed rule changes as a diff or replacement config.
- **Gaps**: user-facing symptoms that no alert covers, especially missing SLO burn-rate alerts.

Don't edit files until the user agrees with the proposed changes.
