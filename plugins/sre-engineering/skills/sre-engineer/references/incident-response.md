# Incident Response

Sources: SRE Book ch.12 "Effective Troubleshooting", ch.13 "Emergency Response", ch.14 "Managing Incidents", ch.16 "Tracking Outages", App. C "Example Incident State Document". SRE Workbook ch.9 "Incident Response".

## Principles

- **Four basics** (Workbook ch.9):
  1. A clear line of command.
  2. Clearly defined roles.
  3. A working record of debugging and mitigation kept as you go.
  4. Incidents declared early and often.
- **The 3Cs: Coordinate, Communicate, Control.** Google's process (IMAG) is based on the firefighters' Incident Command System (ICS), created in 1968.
- **Priorities, in order: stop the bleeding, restore service, preserve evidence for root-causing.** (Book ch.14) "The highest priority is to resolve the issue at hand quickly." (Book ch.13)
- **Mitigate first.** To mitigate, you only need to know *where* the problem is, not *why*. The order is: assess impact, then mitigate, then root-cause, then fix and write the postmortem. (Workbook ch.9)
- It's better to declare early and cancel than to declare late. Spinning up the process is cheap.

## When to declare an incident (Book ch.14)

Declare one if **any** of these is true:
1. You need a second team to help fix the problem.
2. The outage is visible to customers.
3. The issue is still unsolved after **an hour's concentrated analysis**.

These are the author's team's guidelines. Set your own criteria **in advance**, and include severity levels if your organization uses them. Google's books don't define a universal SEV scale.

## Roles (Book ch.14, Workbook ch.9)

| Role | Responsibilities |
|---|---|
| **Incident Commander (IC)** | Holds the high-level state, assigns roles, and **holds every role not yet delegated**. Owns the live incident document. Decides; doesn't debug |
| **Operations lead (Ops)** | Leads the team making changes. **Only the Ops team changes production during the incident** |
| **Communications lead (Comms)** | Is the public face of the response. Sends periodic updates to stakeholders and keeps the incident doc's summary current |
| **Planning lead** | Handles longer-term support: files bugs, arranges handoffs and meals, tracks deviations from normal state so they can be reverted later |

- Ops and Comms report to the IC. The roles scale down, so one person can hold several in a small incident.
- **Handoffs must be explicit**: "You're now the incident commander, okay?" The outgoing IC stays until the new one acknowledges. Write down who the next IC is.
- Rotate people on long incidents. PagerDuty swaps on-call engineers and the IC every 4 hours. (Workbook ch.9)

## Failure modes to avoid (Book ch.14)

- **Sharp focus on the technical problem**: everyone debugs and nobody coordinates or communicates.
- **Poor communication**: stakeholders and the responders themselves don't know the current state.
- **Freelancing**: someone changes production outside the Ops lead's direction, which makes things worse and confuses the record.

## Live incident document (Book App. C, Workbook ch.9)

Start a shared working document as soon as **three or more people** are involved. The IC owns it and the Comms lead keeps its summary current.

```markdown
# Incident: <short title>   Status: ACTIVE | MITIGATED | RESOLVED
Incident Commander: <name> (next IC: <name>) · Ops lead · Comms lead · Planning lead
Declared: <UTC> · Severity (your scale) · Bug / ticket links

## Summary
What is broken, who is affected, current impact, current mitigation.

## Detailed status   (update at least every 4 hours and at every Comms handoff)

## Exit criteria
- Root cause addressed or mitigated, and
- SLIs back within SLO for 30+ minutes (e.g. availability ≥ 99.99%, p99 < 100 ms)

## TODO / action list
- [ ] TODO(owner) ... (link longer-term fixes to bugs)
- [x] DONE ...

## Timeline   (most recent first, UTC, with the person's name)
- 15:36 <name>: shifted traffic to sacrificial cluster; error rate falling
```

## Mitigation playbook

Prepare these **generic mitigations** before you need them, and practice them (Workbook ch.9, ch.8):

| Mitigation | Use when |
|---|---|
| **Roll back** the latest binary, config or flag change | Almost always the first step if something changed. About 70% of outages follow a change, and 68% of outage triggers are binary or config pushes |
| **Drain** a zone, cluster or region | The problem is local |
| **Disable a feature flag** | One new feature is involved |
| **Shed or drop load**, admitting about 1% then ramping up | Overload or a cascading failure (see `overload-cascading.md`) |
| **Add capacity / scale up** | Organic overload. Watch for cold caches and thundering herds |
| **Sacrificial cluster** | Send traffic to one cluster so the others can recover (Book App. C example) |
| **Block traffic** (abuse, a bad client) | One traffic source is the trigger |

In a Workbook case study, a 6h40m outage would have ended by 10 a.m. if the team had rolled back promptly. **Don't wait for the root cause before mitigating.**

## Emergency response lessons (Book ch.13)

- The person who triggered the problem usually knows the most about it. Pull them in without blame.
- **Test the rollback before a risky change or test.** Google now requires a tested rollback before large-scale tests.
- **Canary every change** "regardless of the perceived risk".
- Keep **out-of-band communications and command-line access** that don't depend on the production stack, and practice using them.
- **Sanity-check automation**: "sometimes zero does mean all". An empty filter once matched every machine.
- Ask "what if…?" questions in advance, and test while your best people are present.

## Troubleshooting method (Book ch.12)

1. **Problem report**: expected behavior, actual behavior, how to reproduce. Open a bug for every issue, even ones reported by chat.
2. **Triage**: stop the bleeding. Resist the urge to find the root cause during a major outage. Preserve evidence (logs, core dumps, a copy of bad state).
3. **Examine**: metrics, logs, traces, request tracing, the current state of each component.
4. **Diagnose**: divide and conquer (bisect the request path), ask what, where and why, and check **what changed** most recently.
5. **Test and treat**: test hypotheses in decreasing order of likelihood, weighing the risk of each test. Prefer tests with mutually exclusive outcomes. Watch for side effects and confounders.
6. **Cure**: prove the root cause, fix it, and write it up.

Pitfalls: correlation mistaken for causation, chasing "zebras" before "horses" (unlikely causes before likely ones), assuming the last outage's cause has returned, and confirmation bias.

## Tracking outages (Book ch.16)

- Track **alerts** (single notifications) and **incidents** (groups of alerts) separately, with counts per day and per shift.
- Tag incidents with free-form namespaces (`cause:`, `action:`, `service:`). Compare across teams and over time to find systemic causes.
- Postmortems cover only large incidents, so the tracker also catches frequent small issues.

## Incident checklist

- [ ] Declaration criteria and severity scale agreed in advance
- [ ] Incident channel, document template, 2–3 message templates and contact list ready
- [ ] Roles assigned explicitly, with the IC named and handoffs acknowledged
- [ ] Only the Ops lead's team touches production; changes logged in the timeline
- [ ] Generic mitigations (rollback, drain, flag-off, shed load) documented and practiced
- [ ] Out-of-band access and comms tested
- [ ] Exit criteria defined, e.g. SLIs within SLO for 30+ minutes
- [ ] Postmortem criteria checked at close (see `postmortems.md`)
- [ ] Regular drills (Wheel of Misfortune, DiRT), each followed by a write-up
