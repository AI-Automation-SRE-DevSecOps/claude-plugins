# SRE Workbook, Chapter 17: Identifying and Recovering from Overload

Source: https://sre.google/workbook/overload/ (Peetz, Quesada Torres, Melo, with Bates; Part III, Processes)

## Core principles

- **Definitions (intro).** *Operational load* is "the ongoing maintenance tasks that keep systems and services running". It comes in three types: **pages**, **tickets**, and **ongoing operational responsibilities**. Pages and urgent tickets are called **interrupts**.
- **Operational overload** (also called *work overload*): the team "can't make progress toward key priorities because urgent issues continually preempt project work." Overload also makes engineers more likely to make errors.
- **Perceived overload**: individuals *feel* they have too much work. It usually follows several organizational or work changes in a short period with little chance to talk them through with leadership. It "has the same effects as operational overload", and it turns into objective overload (Case Study 2).
- **Overload is a self-reinforcing cycle (Conclusion).** Interrupts lead to overload, overload creates psychosocial stressors, and those stressors hurt work further.
- **Measure before you change the workload (From Load to Overload).** Don't assume the workload itself has to change. First quantify it (for example, ticket and page counts over time), take a one-time snapshot by asking every member to list all their tasks, then look at psychological stressors.
- **Psychological safety comes first.** "A team can function only as well as its individual members." Giving team members more control reduces perceived overload. Micro-management does not.
- Perceived overload "can't be measured by the amount of toil or operational work" (Conclusion).

## Concrete numbers and thresholds (exact)

| Item | Value | Section |
|---|---|---|
| Cap on operational work | **50% of an engineer's time** ("Google SRE teams cap operational work at 50%") | Intro |
| Overload threshold | "can vary from team to team" | Intro |
| Open tickets per team member | **10 or fewer**. More than 10 triggers tech-lead action | CS1 Lessons Learned |
| Interrupt triage cadence | **once every two weeks** | CS1 Lessons Learned |
| CS1 backlog cleanup | **two days** (1 day intensive triage + 1 day docs/automation) cleared "several months of interrupts" | CS1 Implementation |
| CS1 team loss | **two-thirds** of the team, including the manager, in mid-2016 | CS1 Background |
| CS2 on-call | six or seven engineers per site (Sydney/Zürich). Zürich rotation shrank to **five** | CS2 Background |
| Ticket SLO that backfired | handle tickets within **three days**. Footnote 3: at least **one additional day** of ticket follow-up per shift | CS2 Background |
| Alert silence duration | "typically a day, sometimes up to a week" | CS2 Short-term |
| Example pager storm | "20+ pages" that may just be bad monitoring | From Load to Overload |
| CS2 timeline | short-term actions within **a month**, mid-term within **three months**, little overload left after **about one year** | CS2 Implementation/Effects |
| Planning cadence (CS2) | meet at end of **each quarter** to prioritize | Prioritize and triage within one quarter |

## Recommended practices and procedures

**Case Study 1: Work Overload When Half a Team Leaves.** (The heading says "half". The text says two-thirds.)
- Get the whole team in one room. List *all* responsibilities (project backlog, ops work, tickets) and triage every item.
- Find low-effort automation. Write self-service docs for common customer problems.
- Close obsolete, redundant, or overstated tickets. Fix non-actionable monitoring that generates tickets.
- Before you set non-critical work aside, document your progress on it.
- "When in doubt, we dropped a task" and marked it for a second triage. Almost none of those tasks were resumed.
- Tech lead remedies when someone has more than 10 tickets: remind the team to close stale tickets; offload tickets from overloaded members; prompt individuals to work their queues; run a team-wide **one-day ticket fix-it**; assign work to fix the sources of tickets.

**Case Study 2: Perceived Overload After Organizational and Workload Changes.**
- Short term: a semiregular round table. A **better load metric**: tickets are auto-assigned to the on-caller, who keeps them after the shift, and the metric is the time needed after the shift to resolve them (this replaced page count). Audit and remove alerts that aren't user-facing. Silence alerts generously but with time limits; anything not fixable in minutes gets a tracking ticket. Give the team a dedicated manager, which replaced one shared across three teams. Rebalance the team with experienced outside SREs. Hold social events.
- Mid term: limit ops work to on-call time. **Return one service to its dev team.** Cross-train. Borrow on-callers from the remote team. Backfill the open roles. Tackle each alert as its silence expires, reviewing repetitive or no-action pages in the weekly production meeting. Hold listening events with skip-level managers.
- Long term: align the service SLOs with the backend SLOs, make services uniform, and bring old services up to current production standards.

**Strategies for Mitigating Overload.**
- *Identify and alleviate psychosocial stressors.* Focus on what you control (backlog size, silencing pages). Tell partner dev teams you are overloaded. Assign point people and tech leads per technology. Make decisions transparently and, if possible, democratically.
- *Prioritize and triage within one quarter.* Schedule **interrupt-free time** (no on-call). "If absolutely necessary, drop work."
- *Protect yourself in the future.* Set up workload metrics and review them regularly. Keep a lightweight triage process. During overload, prioritize toil-reducing projects "even more than you would" otherwise. Everyone owns the early warning signs.

## Symptom checklist (Recognizing the Symptoms of Overload)

- [ ] Decreased team morale: rants, negative surveys. Use active listening "without judgment".
- [ ] Long hours and/or working while sick. Leaders should work contractual hours and stay home when sick.
- [ ] More frequent illness.
- [ ] Unhealthy task queue: missed deadlines, or no time for regular queue review.
- [ ] Imbalanced metrics: long time to close a single issue, a high proportion of time on toil, many days to close issues from an on-call session.

The team should choose its metrics together. "There is no one-size-fits-all approach."

## Anti-patterns Google warns against

- The **sunk cost fallacy**: refusing to drop work that is already in progress (CS1).
- Letting automation and root-cause work lose to interrupts forever (CS1).
- A strict ticket SLO that eats into post-shift rest and other ops work (CS2).
- Gradual page creep that nobody notices. Untuned monitoring on newly onboarded services (CS2).
- Never treating "hand back the service" as an option (CS2).
- A manager shared across teams and not on call, who dismisses the team's concerns (CS2).
- Managers who impose a load metric without understanding each person's work habits (Imbalanced metrics).
- Micro-management under stress (Reducing Overload).
- Indefinite alert silences, which "might mask outages" (CS2).

## Short quotes

- "Google SRE teams cap operational work at 50% of an engineer's time."
- "Perceived overload *is*, in fact, overload."
- "each team member should have 10 or fewer open tickets to avoid overload."
- "In general, giving team members more control and power reduces perceived overload."
- "Hope is not a strategy, but it certainly helps team morale."
