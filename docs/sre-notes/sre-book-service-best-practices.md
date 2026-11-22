# Appendix B: A Collection of Best Practices for Production Services

Source: Google SRE Book (2017), https://sre.google/sre-book/service-best-practices/
Author: Ben Treynor Sloss, edited by Betsy Beyer. A short appendix of terse rules, organized into nine sections: Fail Sanely, Progressive Rollouts, Define SLOs Like a User, Error Budgets, Monitoring, Postmortems, Capacity Planning, Overloads and Failure, SRE Teams.

## Core principles

- **Fail sanely (Fail Sanely):** When config input is bad or implausible, keep running on the *previous* state **and** alert. Preserve function "possibly at the expense of being overly permissive or overly simplistic"; wait for a human to approve the new data.
- **Change in stages (Progressive Rollouts):** Nonemergency rollouts *must* be staged, for both config and binary changes.
- **Measure from the user's viewpoint (Define SLOs Like a User):** Measure availability and performance "in terms that matter to an end user."
- **Error budgets balance reliability against velocity (Error Budgets):** They give SRE and dev "a common, data-driven mechanism for assessing launch risk."
- **Human attention is scarce (Monitoring):** Anything that bothers a human must either need action now (page) or be a bug/ticket.
- **Blameless postmortems (Postmortems):** Fix the environment, not people.
- **Degrade, don't fall over (Overloads and Failure):** Overloaded services should return "reasonable but suboptimal results."
- **Cap ops load and share it with dev (SRE Teams).**

## Concrete numbers, thresholds, formulas

| Item | Value as written | Section |
|---|---|---|
| Error budget formula | "1 minus a service's SLO"; e.g. 99.99% target → 0.01% budget for unavailability | Error Budgets |
| Budget period | "over some period; we often use a month" | Error Budgets |
| Quarterly reset | For mature services with SLO "greater than 99.99%", quarterly rather than monthly reset, "because the amount of allowable downtime is small" | Error Budgets |
| What spends budget | "background rate of errors plus any downtime" | Error Budgets |
| Monitoring output types | Exactly three: Pages (act *now*), Tickets (act "within a few days"), Logging (no one needs to look immediately) | Monitoring |
| Capacity | "N + 2": peak traffic handled by N instances (possibly degraded) while the **largest 2** instances are unavailable (one planned + one unplanned outage at once) | Capacity Planning |
| Ops work cap | SRE operational work "no more than 50%" of time; overflow goes to the product dev team | SRE Teams |
| On-call team size | "at least eight people"; if split across two well-separated sites (e.g. California and Ireland), "six people at each site is the minimum" | SRE Teams |
| On-call load | "no more than two events per on-call shift (e.g., per 12 hours)" | SRE Teams |
| Config shrink check | Alert if config is "N% smaller than the previous version" (N unspecified) | Fail Sanely |
| Gmail example | Client-side measurement moved Gmail "from about 99.0% available to over 99.9% available in a few years" | Define SLOs Like a User |
| 2005 DNS incident | Empty DNS file accepted; served NXDOMAIN "for six minutes for all Google properties" | Fail Sanely |
| Mobile client fix latency | Updating client code can take "possibly weeks" | Overloads and Failure |

Note: the N+2 rule tolerates loss of the *largest two* instances, not any two. The on-call minimum is 8 for a single site, but 6 *per site* (12 total) for a two-site team.

## Recommended practices and procedures

**Fail Sanely**
- Validate both syntax and, if possible, semantics of config.
- Watch for empty, partial, or truncated data. For delayed data, "Alert well before the data is expected to expire."
- After 2005 DNS incident: sanity checks on new config (e.g. confirm virtual IPs for google.com present); keep serving previous entries until a new file passes input checks.
- After 2009 incident (a URL list replaced by "/" marked the entire Web as malware): checks for dramatic file-size changes and for matching sites unlikely to contain malware would have stopped it.

**Progressive Rollouts**
- Apply changes to small fractions of traffic and capacity at a time; service size, rollout size, and risk profile set the percentages and time between stages.
- Run different stages in different geographies to catch diurnal-cycle and traffic-mix issues.
- Supervise rollouts by the engineer or, "preferably", "a demonstrably reliable monitoring system".
- On unexpected behavior: "roll back first and diagnose afterward" to minimize MTTR.

**Error Budgets**
- While budget remains, dev is free "(within reason)" to launch features and updates.
- When spent: freeze changes, except urgent security and bug fixes addressing the cause of the increased errors, until the budget is earned back or the month resets.

**Postmortems**
- Assume people were intelligent, well intentioned, and making the best choices with the information they had.
- Remedies: system design that avoids entire classes of problems, easy access to the right information, automatic validation of operational decisions.

**Capacity Planning**
- Validate past demand forecasts against reality until they consistently match. Divergence means unstable forecasting, inefficient provisioning, and shortfall risk.
- Use load testing, "rather than tradition", to set the resource-to-capacity ratio. Re-check it as the system changes.
- Don't mistake day-one (launch) load for steady-state load.

**Overloads and Failure**
- Degrade gracefully. Example: Search serves a smaller fraction of the index and drops features like Instant. Search SRE tests clusters *beyond rated capacity*.
- If even degraded responses are too expensive: graceful load shedding with well-behaved queuing and dynamic timeouts. Other options are tarpitting (answering after a significant delay) and sending errors to a consistent subset of clients.
- Respond to cascading failures by dropping a fraction of traffic "(including retries!)" upstream once total load exceeds total capacity.
- "Every client that makes an RPC must implement exponential backoff (with jitter) for retries."

**SRE Teams**
- Include product developers in on-call and ticket rotation even without overflow, which gives them an incentive to reduce toil.
- Hold a regular production meeting between SRE and dev.
- Routinely practice hypothetical outages (Disaster Role Playing) and improve incident docs while doing so, because rare incidents leave teams out of practice.

## Anti-patterns Google warns against

- Accepting empty, truncated, or drastically changed config without question (the 2005 and 2009 incidents).
- Measuring availability only at the server, which overstated Gmail's availability.
- Big-bang, unmonitored rollouts, and diagnosing before rolling back.
- Sending alerts to email: "the moral equivalent of piping them to /dev/null". It is "an attractive nuisance" that relies on "eternal human vigilance", so the eventual outage is worse.
- Blaming or "fixing" people.
- Sizing capacity by tradition or stale load ratios, or by launch-day traffic.
- Retries without backoff or jitter. Mobile clients are especially risky because there are millions of them and fixes are slow to ship.
- More than two events per shift. This signals problems with design, monitoring sensitivity, or follow-through on postmortem bugs.

## Checklists

Config change gate: syntax valid → semantics valid → not empty/truncated → size delta under N% → semantic plausibility (e.g. required VIPs present, not matching everything) → otherwise keep the old config and alert.

Alert triage: needs human action now → page. Needs action within days → ticket/bug. Otherwise → log. Never email-only.

## Verbatim quotes

- "Fail in a way that preserves function, possibly at the expense of being overly permissive or overly simplistic."
- "If unexpected behavior is detected, roll back first and diagnose afterward in order to minimize Mean Time to Recovery."
- "A budget is simply 1 minus a service's SLO"
- "If it's important enough to bother a human, it should either require immediate action (i.e., page) or be treated as a bug"
- "we can't 'fix' the people, but must instead fix their environment"
- "Services should produce reasonable but suboptimal results if overloaded."
