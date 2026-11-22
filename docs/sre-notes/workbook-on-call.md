# The Site Reliability Workbook, Chapter 8: On-Call

Source: https://sre.google/workbook/on-call/. Authors: Ollie Cook, Sara Smollett, Andrea Spadaccini, Cara Donnelly, Jian Ma, Garrett Plasky (Evernote), with Stephen Thorne and Jessie Yang. This chapter follows up on Chapter 11, "Being On-Call", of the first SRE book.

## Core principles
- **Definition**: being on-call means "being available during a set period of time, and being ready to respond to production incidents during that time with appropriate urgency."
- **Goal**: cover critical services, and "never achieve reliability at the expense of an on-call engineer's health" (Recap section).
- **Balance**: SREs spend **at least 50% of their time on project work**. Teams must have enough staff to make that possible.
- **Psychological safety** is vital. On-call engineers need procedures and escalation paths behind them. "The on-call engineer is never alone."
- **Compensation**: out-of-hours work should be paid for. Google offers time-off-in-lieu or cash, capped at a proportion of salary. The cap stops engineers taking on too many shifts for economic reasons.
- **Alerts must be immediately actionable**, with a high signal-to-noise ratio. Paging should be SLO-based and symptom-based (Alerting).
- **Flexibility is a principle**, not a fixed set of rules (On-Call Flexibility).
- **SRE owns production**, unlike traditional ops. Feature developers are collaborators but don't own reliability work (Conclusion; Proposal one).

## Concrete numbers, thresholds and formulas (as written)
- **Pager budget**: "a maximum of two incidents per on-call shift." Footnote 2: one incident is one "problem," no matter how many alerts fired for it, and **one shift is 12 hours**. The Alerting section says it again: "maximum of two distinct incidents per 12-hour shift."
- **Pager load definition**: "the number of paging incidents that an on-call engineer receives over a typical shift length (such as per day or per week). An incident may involve more than one page."
- **New-hire readiness**: most Google SRE teams don't expect new hires to be on-call before **3 to 9 months**. In the Mountain View case the team shadowed at 2 months and was primary at 3 months.
- **Availability math**: 99.98% is about **26 minutes of downtime per quarter**. 99.99% gives about **15 minutes of error budget per quarter**, which is why you roll back instead of rolling forward. 99.999% gives **80 seconds per quarter**, which may need self-healing.
- **Mountain View case**: machine costs cut by 40%. Training checklist of about two dozen focus areas.
- **Connection team**: budget of 2 paging incidents per shift, but they actually got 5 per shift for a year. One-third of shifts went over budget. Human error was the second most common cause of new bugs.
- **Break-even formula**: a project of 3 working weeks (120 hours), with each page costing about 4 hours to handle, breaks even after **30 pages** (120 / 4).
- **Testing new alerts**: run them in test mode for "a week of testing is probably about right." The right length depends on the alert and the system.
- **Trend tracking**: a **21-day trailing average** of pager load.
- **Shift length**: limit shifts to **12 hours**. 24 hours without a break is not sustainable. Another option is **3 days on, 4 days off**.
- **Minimum staffing for 24/7**: **5 per site** (multisite) or **8** (single-site). Add one extra per site to cover absences, giving **6 per site (multisite) or 9 (single-site)**.
- **Pager response vs commute**: with a 5-minute response SLO and a 30-minute commute, someone else must cover while you travel.
- **Evernote**: migration took 70 days and moved 3.5 PB of data.

### Table 8-1. Examples of realistic response times
| Incident | Response time | SRE impact |
|---|---|---|
| Revenue-impacting network outage | 5 minutes | Within arm's reach of a charged, authenticated laptop with network access; cannot travel; coordinate heavily with secondary |
| Customer order batch processing stuck | 30 minutes | Can do a quick errand or short commute; secondary need not cover |
| Pre-launch service DB backups failing | Ticket (work hours) | None |

### Evernote event classes (Restructuring our monitoring and metrics)
- **P1**: deal with it immediately. It is actionable, pages the on-call, leads to triage, and affects the SLO.
- **P2**: next business day. Usually not customer-facing. Goes to team email and the event-stream channel.
- **P3**: informational only. Shows up in dashboards and passive email, including capacity planning data.
- P1 and P2 events each get an incident ticket. Pages are triaged as **Sev 1 to 3**. Sev 1 has fixed escalation criteria, an incident manager, a scribe, a comms lead and an automatic postmortem. For Sev 2 and 3 the on-call handles the incident and writes a shorter postmortem.

## Recommended practices and procedures
- **Shift duties** (Training roadmap): read the previous shift's handoff at the start, minimize user impact first and then fully address the issue, and send a handoff email at the end.
- **Playbooks**: create a playbook entry for every alert. If a playbook is a fixed list of commands, automate it. Agree with your team on the minimum structured details every entry must have.
- **Training**: starter projects, codelabs, lab sessions, deep dives, Wheel of Misfortune exercises (recorded), shadowing, and the previous team acting as backup.
- **Pager load inputs** (Pager load inputs): these fall into three groups.
  - *Production bugs*: existing bugs, new bugs, how fast bugs are identified, and how fast they are mitigated.
  - *Alerting*: thresholds, new alerts, and whether the service's SLOs line up with the SLOs of its dependencies.
  - *Human processes*: rigor of follow-up, data quality, attention to trends, and manual changes to production.
- **New bugs**: use canaries and staging, do load testing, and "detect, roll back, fix, and roll forward." Fix existing bugs before building new features. Make production changes through automation driven by intent config.
- **Identification delay**: link pages to consoles, run Wheel of Misfortune, ship small releases, keep a searchable change log, and ask for help.
- **Mitigation delay**: roll back, use feature isolation (flags), and drain traffic away from bad parts of the system. Avoid changes you can't roll back, such as API-incompatible changes and lockstep releases.
- **New alerts**: the whole team reviews them. Test them in production without paging first, for example by emailing the alert's author. Use the trigger rate to predict how much pager budget the alert will use, and explicitly approve or reject it as a team.
- **Follow-up**: find the root cause of every page. If you can't, add monitoring or logging. Prefer **systemic** or **monitoring (prevention)** fixes over **point** fixes. The on-caller does no project work during the shift and works on system-health bugs instead.
- **Data quality**: file a placeholder bug for each page and link pages to bugs. Rank bugs by how many pages they cause. Set nonpaging alerts for pages that weren't handled to the team's standard.
- **Vigilance**: review pager load trends at production meetings. Set ticket alerts to tech leads or managers when load crosses a warning threshold. Hold regular meetings between SRE and developers.
- **Scheduling tool**: it should rearrange and rebalance shifts, take preferences and past load into account, and **never change an already generated schedule**. Peer review of swaps is a good tradeoff. Give part-timers proportionally less on-call work.
- **Team dynamics**: give ops/SRE engineers real ownership, have SREs change the code themselves and send reviews to subject experts, use a "fun budget," and seat the whole rotation together.

## Anti-patterns
- Paging for things that could be automated or handled by a ticket.
- Answering pages by **relaxing thresholds**. Under SLO- and symptom-based alerting this is "rarely an appropriate response".
- Writing off pages as "transient" or "fixed itself", or concluding "cause unknown".
- Only making point fixes.
- "Roll forward, fix, and roll forward again."
- 24/7 shifts with no break, or on-call by a single person for a whole week.
- Ignoring duplicate pages ("survive the week" culture). Alerting on error *rate* instead of error *ratio*.
- Renaming ops to SRE with no real change in responsibilities (footnote 1; Proposal one).

## Checklist: follow-up questions (Rigor of follow-up)
1. How can I prevent this specific bug from happening again?
2. How can I prevent bugs like this, in this system and in other systems I'm responsible for?
3. What tests could have stopped this bug before production?
4. What ticket alerts could have triggered action before it paged?
5. What informational alerts could have shown it on a console earlier?
6. Have I maximized the impact of my fixes?

## Short verbatim quotes
- "We target a maximum of two incidents per on-call shift, to ensure adequate time for follow-up."
- "If your playbooks are a deterministic list of commands... we recommend implementing automation."
- "relaxing alert thresholds is rarely an appropriate response to being paged."
- "You should rarely conclude that a page is triggered by 'cause unknown.'"
- "it must never change an already generated schedule."
- "In our experience, 24 hours of on-call duty without reprieve isn't a sustainable setup."
