# Source Map

Every reference file is a summary of Google's two free online books. Cite the chapter when you give a number or rule, and point users to the original for depth.

- *Site Reliability Engineering* (2016): https://sre.google/sre-book/table-of-contents/
- *The Site Reliability Workbook* (2018): https://sre.google/workbook/table-of-contents/

## SRE Book

| Ch. | Title | Reference file |
|---|---|---|
| 1 | Introduction | SKILL.md, toil-automation, capacity-launch |
| 2 | The Production Environment at Google | capacity-launch |
| 3 | Embracing Risk | slo-sli, error-budget-policy |
| 4 | Service Level Objectives | slo-sli |
| 5 | Eliminating Toil | toil-automation |
| 6 | Monitoring Distributed Systems | monitoring, alerting |
| 7 | The Evolution of Automation at Google | toil-automation |
| 8 | Release Engineering | release-config |
| 9 | Simplicity | team-engagement |
| 10 | Practical Alerting from Time-Series Data | monitoring, alerting |
| 11 | Being On-Call | on-call |
| 12 | Effective Troubleshooting | incident-response |
| 13 | Emergency Response | incident-response, resilience-testing |
| 14 | Managing Incidents | incident-response |
| 15 | Postmortem Culture: Learning from Failure | postmortems |
| 16 | Tracking Outages | incident-response |
| 17 | Testing for Reliability | resilience-testing, release-config |
| 18 | Software Engineering in SRE | capacity-launch |
| 19 | Load Balancing at the Frontend | overload-cascading |
| 20 | Load Balancing in the Datacenter | overload-cascading |
| 21 | Handling Overload | overload-cascading |
| 22 | Addressing Cascading Failures | overload-cascading |
| 23 | Managing Critical State: Distributed Consensus | data-state |
| 24 | Distributed Periodic Scheduling with Cron | data-state |
| 25 | Data Processing Pipelines | data-state |
| 26 | Data Integrity | data-state, resilience-testing |
| 27 | Reliable Product Launches at Scale | capacity-launch, release-config |
| 28 | Accelerating SREs to On-Call and Beyond | on-call |
| 29 | Dealing with Interrupts | team-engagement, on-call |
| 30 | Embedding an SRE to Recover from Operational Overload | team-engagement |
| 31 | Communication and Collaboration in SRE | team-engagement |
| 32 | The Evolving SRE Engagement Model | capacity-launch, team-engagement |
| 33 | Lessons Learned from Other Industries | postmortems, resilience-testing |
| 34 | Conclusion | team-engagement |
| App. A | Availability Table | slo-sli |
| App. B | Best Practices for Production Services | error-budget-policy, monitoring, release-config |
| App. C | Example Incident State Document | incident-response |
| App. D | Example Postmortem | postmortems |
| App. E | Launch Coordination Checklist | capacity-launch |
| App. F | Example Production Meeting Minutes | team-engagement, error-budget-policy |

## SRE Workbook

| Ch. | Title | Reference file |
|---|---|---|
| 1 | How SRE Relates to DevOps | team-engagement, toil-automation |
| 2 | Implementing SLOs | slo-sli, error-budget-policy |
| 3 | SLO Engineering Case Studies (Evernote, Home Depot) | slo-sli, monitoring |
| 4 | Monitoring | monitoring |
| 5 | Alerting on SLOs | alerting |
| 6 | Eliminating Toil | toil-automation |
| 7 | Simplicity | team-engagement |
| 8 | On-Call | on-call, alerting |
| 9 | Incident Response | incident-response |
| 10 | Postmortem Culture: Learning from Failure | postmortems |
| 11 | Managing Load | overload-cascading, capacity-launch |
| 12 | Introducing Non-Abstract Large System Design | capacity-launch |
| 13 | Data Processing Pipelines | data-state |
| 14 | Configuration Design and Best Practices | release-config |
| 15 | Configuration Specifics | release-config |
| 16 | Canarying Releases | release-config |
| 17 | Identifying and Recovering from Overload | team-engagement, alerting |
| 18 | SRE Engagement Model | capacity-launch, team-engagement |
| 19 | SRE: Reaching Beyond Your Walls | slo-sli, team-engagement |
| 20 | SRE Team Lifecycles | team-engagement, on-call |
| 21 | Organizational Change Management in SRE | team-engagement |
| Concl. | Conclusion | team-engagement |
| App. A | Example SLO Document | slo-sli |
| App. B | Example Error Budget Policy | error-budget-policy |
| App. C | Results of Postmortem Analysis | postmortems, release-config |

## How to use these sources

- Google's numbers are **defaults and examples from Google's context**. Where a number is an illustration (canary percentages, bucket SLOs, case-study figures), the reference file says so.
- If the user's situation differs (a small team, low traffic, a managed platform), keep the principle, adapt the number, and say so explicitly.
- The books predate some modern tooling (OpenTelemetry, Kubernetes-native SLO operators). Translate the concepts; don't invent new Google rules.
