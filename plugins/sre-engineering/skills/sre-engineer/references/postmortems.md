# Postmortems

Sources: SRE Book ch.15 "Postmortem Culture: Learning from Failure", ch.33 "Lessons Learned from Other Industries", App. D "Example Postmortem". SRE Workbook ch.10 "Postmortem Culture", App. B, App. C "Results of Postmortem Analysis".

## Principles

- **Blameless**: look for contributing causes without blaming any person or team. Assume everyone acted with good intentions on the information they had. Ask "what", not "who". (Book ch.15)
- **The main goal is preventive action**, not documentation. "An unreviewed postmortem might as well never have existed."
- A postmortem is Google's form of **CAPA** (corrective and preventive action). It answers four questions: what happened, how well the response worked, what to do differently, and which actions prevent a repeat. (Book ch.33)
- **Near misses count.** Treat them as preemptive postmortems. Include a "Where we got lucky" section.
- **Reward closing action items**, not just writing the document.

## When to write one (Book ch.15)

Agree the criteria **before** incidents happen. Common triggers:
1. User-visible downtime or degradation beyond a threshold
2. Data loss of any kind
3. On-call intervention, such as a rollback or rerouting traffic
4. Resolution time above a threshold
5. A monitoring failure, usually meaning a human found the problem before monitoring did

Any stakeholder may also ask for one. Policy-driven triggers from the error budget policy (Workbook App. B):
- A single incident consuming **more than 20% of the error budget over four weeks** requires a postmortem with **at least one P0 action item**.
- After any user-affecting outage, Google's standard is that the postmortem has **at least one P0 or P1 bug** (Workbook ch.10).

## Trigger vs root cause

- **Trigger**: the event that set the incident off, such as a binary push, config push, traffic spike or hardware failure.
- **Root cause**: the defect that, once fixed, gives confidence the event "won't happen again in the same way". There can be several, and each should be fixed.
- Example (Book App. D): the **root cause** was a latent file-descriptor leak plus a cascading failure under load. The **trigger** was an 88× traffic spike.
- Google's 2010–2017 analysis (Workbook App. C) found these top **triggers**: binary push 37%, configuration push 31%, user behavior change 9%, processing pipeline 6%, service provider change 5%, performance decay 5%, capacity management 5%, hardware 2%. The top **root-cause categories**: software 41%, development process failure 20%, complex system behavior 17%, deployment planning 7%, network failure 3%. Don't call 37% and 31% "root causes"; they are triggers.

## Action items (Workbook ch.10)

Every action item needs:
- **One owner**. There can be collaborators, but there must be one owner.
- **A tracking bug** and **a priority** (P0, P1, P2…).
- **A type**: *mitigate* (reduce impact or duration next time), *prevent* (stop it happening), *process*, or *other*. Include at least one prevent item, not only mitigations.
- **A verifiable end state**: "Add a load test that fails if fd count grows > 10%/h", not "Improve monitoring".

Reject vague or blaming items: "be more careful", "train humans", "improve X", "investigate". Avoid **knee-jerk** items, such as one specific alert for this exact failure, when a unit test, a systemic fix or an SLO alert would cover the whole class. (Book App. D footnotes)

## Process

1. **Draft** within days, while memories and logs are fresh. Google's good example was out less than a week after the incident closed; the bad one took four months.
2. **Collaborate**: use a shared document with comments, and include the people who responded.
3. **Review**: a senior engineer checks completeness. Were the impact and root cause captured, are the action items appropriate and prioritized, and were findings shared with stakeholders?
4. **Publish**: open to the whole company by default, in a searchable repository with machine-readable tags (service, trigger, root-cause category).
5. **Follow up**: track action items to closure, and review open postmortem bugs in the production meeting.
6. **Learn at scale**: run monthly "postmortem of the month" readings, reading clubs, Wheel of Misfortune sessions built from real postmortems, FixIt weeks (twice a year), and weekly outage reports for leadership. Do trend analysis across postmortems.

## Writing style

- Neutral, factual and blameless. No names attached to mistakes ("the operator", or role names). No dramatic wording ("careless", "!!!", "obviously").
- Quantify impact: duration, users or requests affected, error budget consumed, revenue impact, data loss.
- Record timestamps in UTC, and keep the outage window separate from the incident window. For example, the outage ran 14:54–16:00 while the incident ran 15:01–16:30.

## Template (based on Book App. D and Workbook ch.10)

```markdown
# Postmortem: <title>   (Blameless)
Date · Authors · Status (Draft | In review | Final) · Reviewers · Incident doc / bug links

## Summary
2–4 sentences: what happened, user impact, how it was mitigated.

## Impact
- Duration: outage <start>–<end> UTC (<n> min); incident <start>–<end> UTC
- Users / requests affected: <number or %>, by region and request class
- Error budget consumed: <x>% of the 4-week budget  (> 20% → P0 action item required)
- Revenue / data impact: <estimate or "none">

## Root causes
The latent defects that, once fixed, prevent recurrence. There may be several.

## Trigger
The event that exposed them, e.g. a config push, binary release or traffic change.

## Detection
How and when it was detected (alert name, or a human report). Time from start to detection.
Did the right alert fire? Would SLO burn-rate alerting have caught it sooner?

## Resolution
What mitigated it and what fixed it. Which generic mitigations (rollback, drain, flag) were used or missed.

## Action items
| Action | Type (mitigate / prevent / process / other) | Priority | Owner | Bug |
|---|---|---|---|---|
| Add an fd-leak regression test to CI | prevent | P0 | <one owner> | BUG-123 |
| Make rollback of config X one command, and document it | mitigate | P1 | <one owner> | BUG-124 |

## Lessons learned
### What went well
### What went wrong
### Where we got lucky

## Timeline (UTC)
| Time | Event |
|---|---|

## Supporting information
Graphs, logs, links.
```

## Review checklist

- [ ] Meets an agreed trigger, or was requested; budget impact calculated
- [ ] Blameless language throughout
- [ ] Trigger and root causes kept separate; contributing factors listed
- [ ] Detection analysed: what fired, how long it took, and what should have fired
- [ ] At least one prevent item; every item has one owner, a priority, a bug and a verifiable end state
- [ ] If more than 20% of the budget was used, at least one P0 item
- [ ] Went well / went wrong / got lucky all filled in
- [ ] Reviewed by a senior engineer and published openly with tags
- [ ] Action items tracked to closure in the production meeting
