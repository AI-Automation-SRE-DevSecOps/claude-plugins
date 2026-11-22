# On-Call

Sources: SRE Book ch.11 "Being On-Call", ch.28 "Accelerating SREs to On-Call and Beyond", ch.29 "Dealing with Interrupts", App. B. SRE Workbook ch.8 "On-Call", ch.20 "SRE Team Lifecycles".

## Principles

- On-call means "being available during a set period of time, and being ready to respond to production incidents during that time with appropriate urgency."
- "Never achieve reliability at the expense of an on-call engineer's health." Psychological safety matters: "the on-call engineer is never alone". Clear escalation paths and procedures back them up.
- On-call is one of the things SREs do, not the whole job. It also shows whether someone is ready for production work.
- Pay for out-of-hours on-call, as time off in lieu or cash. Google caps it at a proportion of salary so that people don't take extra shifts for the money.

## Time budget (Book ch.11)

| Activity | Share of SRE time |
|---|---|
| Engineering / project work | **≥ 50%** |
| On-call | **≤ 25%** of total time |
| Other operational work (tickets, manual tasks) | the rest of the ≤ 50% ops cap |

When ops work goes over 50%, send the excess back to the development team (tickets, pages, on-call share) until it drops. (Book ch.1)

## Load

- **At most 2 incidents per 12-hour shift.** An *incident* is one underlying problem, however many alerts or pages it produces. Handling one incident, including root cause, remediation and postmortem, takes about 6 hours. (Book ch.11, Workbook ch.8)
- **Median pages per day should be about 0.**
- **Underload is a risk too.** Everyone should be on call at least once or twice a quarter so they stay familiar with production.
- Track **pager load as a 21-day trailing average**, and review it at the weekly production meeting. When it crosses a warning threshold, open a ticket for the tech lead or manager. (Workbook ch.8)
- **Break-even check**: a 3-week project (120 h) that removes a page costing about 4 h of handling pays for itself after 30 pages.

## Staffing

| Setup | Minimum | With slack for vacation and illness (Workbook ch.8) |
|---|---|---|
| Single site, 24/7, primary + secondary, week-long shifts | **8 engineers** (Book ch.11, App. B) | 9 |
| Two sites, follow-the-sun | **6 per site** (Book) / bare minimum 5 per site (Workbook) | 6 per site |

- **Two sites, 6–8 hours apart, are better than three.** Each site works 12-hour shifts and avoids roughly 12–6 a.m. local time. (Workbook ch.20)
- **Shift length: at most 12 hours.** "24 hours of on-call duty without reprieve isn't a sustainable setup." A 3-days-on/4-days-off pattern is another option.
- **Have one primary.** A single named primary avoids the bystander effect. The secondary is a fallback, or a second responder for interrupts. (Book ch.29)
- **Wait 3–6 months** after splitting a team before splitting its on-call rotation. (Workbook ch.20)
- Never rely on one person to cover a whole week alone.
- **Scheduling tools** should rebalance shifts and respect preferences and past load. They must never change a schedule that has already been generated. Peer-reviewed swaps are fine.

## Response times (Workbook Table 8-1, Book ch.11)

| Incident example | Response time | What it means for the on-call engineer |
|---|---|---|
| Revenue-affecting outage of a user-facing service | **5 minutes** | Laptop charged, authenticated and online, within arm's reach. No travel. Coordinate closely with the secondary |
| Customer batch processing stuck | **30 minutes** | Short errands or a commute are OK |
| Pre-launch service's DB backups failing | **Ticket** (working hours) | None |

At 99.99%, a quarter allows only about 13–15 minutes of downtime, so the response time has to fit inside the error budget. With 99.999% (about 80 s per quarter), you may need self-healing rather than human response. (Book ch.11, Workbook ch.8)

## During a shift

- **Start of shift**: read the previous handoff.
- **During**: minimize user impact first, then fully resolve the problem. The on-caller does no project work, and spends spare time on system-health bugs instead.
- **End of shift**: send a handoff email with ongoing issues, notable alerts and pending follow-ups. (Workbook ch.8, Book ch.16)
- **Prefer rolling back**: "detect, roll back, fix, and roll forward". Avoid "roll forward, fix, and roll forward again".
- **Escalate early.** Bring in the secondary, subject-matter experts or the incident process, and declare an incident when the criteria are met (see `incident-response.md`).
- Follow up on every page with six questions (Workbook ch.8):
  1. How can I prevent this exact bug?
  2. How can I prevent bugs like it, here and in other systems?
  3. Which tests would have caught it before production?
  4. Which ticket alert would have triggered action before it paged?
  5. Which informational alert or console would have shown it earlier?
  6. Have I maximized the impact of the fix (systemic or prevention fixes rather than point fixes)?

## Reducing pager load (Workbook ch.8)

| Source | Levers |
|---|---|
| **Production bugs** | Canaries and staging, load tests, fix existing bugs before building new features, make production changes through automation driven by intent-based config |
| **Slow identification** | Link pages to consoles and dashboards, Wheel of Misfortune practice, small releases, a searchable change log |
| **Slow mitigation** | Rollbacks, feature flags, draining. Avoid changes that can't be rolled back (incompatible APIs, lockstep releases) |
| **Alerting** | SLO and symptom-based alerts, team review of new alerts, an approval process tied to the pager budget, deleting unactionable alerts |
| **Human process** | Rigorous follow-up, data quality (a bug for every page), watching trends, fewer manual production changes |

When pager load stays over budget for a long time (for example 5 incidents per shift instead of 2), apply the policy: hand work back to the developers, freeze features until the load is fixed, or split the rotation. **Relaxing thresholds is "rarely an appropriate response"**.

## Onboarding new on-callers (Book ch.28, Workbook ch.8)

- Most Google teams **don't put new hires on call for 3–9 months**.
- **Avoid "trial by fire".** Don't learn on-call by draining a ticket queue.
- Build a **learning checklist** for each area: who the experts are, key docs, what to know before moving on, and questions to check understanding. Grant access in steps, from read-only up to changing production.
- Practice with **Wheel of Misfortune**: weekly, 30–60 minutes, a game master and two players acting as primary and secondary. Record the sessions.
- Run **break-it exercises**: predict, then break a test system, then check the prediction. Google's example is "Let's burn a search cluster to the ground!", run quarterly.
- **Shadow** first (pages copied to the trainee, business hours at first), then **reverse-shadow**: the trainee is primary and the expert observes without acting.
- New team members update one or two outdated checklist sections and co-author postmortems.

## On-call health checklist

- [ ] At least 50% project time for each engineer, measured each quarter
- [ ] At most 2 incidents per 12-hour shift (21-day trailing average reviewed weekly)
- [ ] Rotation at or above the staffing minimum, with shifts of 12 hours or less, primary and secondary defined
- [ ] Response-time SLO for each alert class, consistent with the error budget
- [ ] A playbook for every page; fixed-command playbooks automated
- [ ] Handoff at every shift change; every page linked to a bug
- [ ] New alerts trialled for about a week and approved against the pager budget
- [ ] On-call compensation and escalation paths documented
- [ ] Training path: checklist, then Wheel of Misfortune, then shadow, then reverse-shadow, then primary
