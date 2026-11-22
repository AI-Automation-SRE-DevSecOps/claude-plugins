# SRE Teams, Engagement, Interrupts and Simplicity

Sources: SRE Book ch.1, ch.9 "Simplicity", ch.29 "Dealing with Interrupts", ch.30 "Embedding an SRE to Recover from Operational Overload", ch.31 "Communication and Collaboration in SRE", ch.32 "The Evolving SRE Engagement Model", ch.34 "Conclusion". SRE Workbook ch.1 "How SRE Relates to DevOps", ch.7 "Simplicity", ch.17 "Identifying and Recovering from Overload", ch.18 "SRE Engagement Model", ch.20 "SRE Team Lifecycles", ch.21 "Organizational Change Management in SRE".

## What makes it SRE (Workbook ch.20, ch.1)

There are three principles:
1. **SLOs with consequences.** The error budget policy actually changes priorities.
2. **Time to make tomorrow better than today.** At least 50% of time goes to engineering.
3. **The team can regulate its own workload.** It can push work back, hand services back, or decline them.

Related practices:
- Product teams own their services by default. Most services never get SRE support. SRE takes a service on after a PRR, and can withdraw support from services that are "irredeemably operationally difficult".
- **Don't just rename Ops to SRE.** That "rename-and-shame" pattern doesn't work without the principles above.
- **Parity of esteem**: rate SREs and developers in roughly the same ways, with the same incentives and shared tools.
- **Staffing ratios**: SREs are kept below 10% of developers (Workbook ch.18). The ratio ranges from about 1:5 for infrastructure to 1:50 for consumer microservices, and is usually near 1:10 (Workbook ch.20).
- **Team size shouldn't grow with service size.** Scale through automation and abstraction. "A 747 still has just two pilots." (Book ch.18, ch.34)
- **You can adopt SRE gradually**, without a large global team first. Order still matters: SLOs and the 50% cap come before everything else. (Workbook Conclusion)

## Team lifecycle and structure (Workbook ch.18, ch.20)

- Keep new hires to **fewer than a third** of the team.
- **Organize teams by technology** rather than by org chart, so reorganizations cause less churn. Shard an overloaded team instead of creating a new one.
- **Avoid single-person and single-site teams.** Use two sites 6–8 hours apart, rotate site duties about yearly, and review team leads at least yearly.
- **Engagement isn't permanent.** SRE can lower the SLO, move work elsewhere, or hand the service back. Being unreliable isn't on its own a reason to get SRE support.
- Lifecycle phases: Architecture and Design, Active Development, Limited Availability, General Availability, Deprecation, Abandoned, Unsupported. **Define SLOs before GA.**

## Interrupts (Book ch.29)

- There are three kinds: **pages** (response measured in minutes), **tickets** (hours to weeks), and **ongoing operational work**.
- **Context switches are expensive.** "A 20-minute interruption … entails two context switches" and can cost "a couple hours" of productive work.
- **Separate interrupt work from project work.** Ideally a person spends a whole week on one or the other; at minimum, a whole day or half-day. The person on call does no project work that week.
- **Make one person, a rotating ticket owner, handle tickets.** Don't assign them randomly or spread them across the team. Add a second person if the volume needs it.
- **Reduce the load**: review tickets regularly for root causes, silence an interrupt until its fix is due, and use policy to send work back to customers through self-service.

## Operational overload (Book ch.30, Workbook ch.17)

- **Overload** means the team can't make progress on its key priorities because urgent work keeps taking precedence. "Perceived overload *is*, in fact, overload."
- **Measure first.** Look at ticket and page trends, and make a one-time inventory of everyone's tasks.
- **Fixes:**
  - Keep each person to **10 or fewer open tickets**, and triage every two weeks.
  - Limit silences to "typically a day, sometimes up to a week".
  - Drop work that doesn't affect the SLO.
  - Hand services back to their developers.
  - Avoid strict ticket SLOs that backfire. A three-day ticket SLO did.
- **Embedding an SRE** (Book ch.30): add one SRE to the overloaded team, not two, since two can make the team defensive. Their jobs:
  - Write an SLO first. Without one, "no other advice in this chapter will be helpful".
  - Find the "kindling": the 2–3 issues most likely to cause the next fire.
  - Own the next blameless postmortem.
  - Sort the fires into toil, which should be automated, and acceptable overhead.
  - Explain the reasoning behind each decision so the team can make it without you.
  - Finish with an after-action report.

## Communication (Book ch.31)

- **The weekly production meeting** runs **30–60 minutes**, covers services rather than people's status updates, and is **compulsory for the whole SRE team**, with dev partners invited. Rotate the chair.
- Standing agenda (Book App. F):
  1. Announcements
  2. Previous action items
  3. Outage review
  4. Paging events
  5. Non-paging events
  6. Monitoring changes and silences
  7. Planned production changes
  8. Resources
  9. Key service metrics (SLO status OK or BAD, value against target, trailing window)
  10. Discussion
  11. New action items, each written `TODO(owner)`
- Sort non-paging events three ways: should have paged (fix the monitoring), needs attention but not a page (keep it as a ticket), or needs no attention (delete it).
- Split cross-site projects into components that one site owns end to end.

## Simplicity (Book ch.9, Workbook ch.7)

- SREs **push back against accidental complexity** and keep removing it from the systems they own. Essential complexity comes with the problem. Accidental complexity "can be resolved with engineering effort".
- **"Every new line of code written is a liability."** Delete dead code. Don't comment it out or leave it behind a flag that's always off (Knight Capital lost $440M that way).
- **Use minimal, versioned APIs** and loose coupling. Avoid "util" or "misc" binaries. Keep data formats backward- and forward-compatible.
- **Measure complexity by proxy**: how long it takes to train someone until they can go on call, how long it takes to explain the system, how many different admin methods and config variants exist, and how old the system is (Hyrum's Law).
- **"Simplification is a feature."** Celebrate removing code like a launch. When diagramming a design, look for retry amplification and **cyclic dependencies**, which can make a cold start impossible. Allowlist dependencies explicitly.
- **Judge a rewrite against the current system improved by the same effort**, not against the current system as it stands.

## Organizational change (Workbook ch.21)

- **Kotter's 8 steps** suit SRE adoption, though urgency usually already exists. **ADKAR** suits coordinating change across distributed teams.
- Migrations must cost adopters "nearly zero". Measure success by adoption first, because a good product doesn't win users on its own.
- Migrate low-traffic, high-importance flows first, writing to both the old and new systems.

## Working with customers and platforms (Workbook ch.19)

- **Once machines call your API, you're a platform**, and reliability becomes a partnership with your customers.
- Up to half of what customers alert on doesn't affect their SLOs. Help them alert on SLOs instead, and build shared dashboards.
- Rank design-review issues by how much error budget each would consume. Run joint postmortems with customers.
