# SRE Book, Chapter 1: Introduction

Source: https://sre.google/sre-book/introduction/. Written by Benjamin Treynor Sloss (VP, Google Engineering, founder of Google SRE). Edited by Betsy Beyer. Epigraph: "Hope is not a strategy."

## Core principles

- **SRE definition** (*Google's Approach to Service Management*): "SRE is what happens when you ask a software engineer to design an operations team." SRE hires software engineers to build systems that do work sysadmins would otherwise do by hand.
- **Engineering focus is mandatory**: without constant engineering, ops load grows and a traditional ops team "scales linearly with service size." SRE headcount should scale **sublinearly** with system size.
- **Aim for "automatic", not just "automated"**: the goal is a service that "basically runs and repairs itself."
- **100% is the wrong reliability target** for nearly everything (*Pursuing Maximum Change Velocity…*). Exceptions named: pacemakers and anti-lock brakes. Users can't tell 100% from 99.999% because the things between them and the service (laptop, home WiFi, ISP, power grid) are collectively much less available than that.
- **The reliability target is a product/business decision, not a technical one.** "The business or the product must establish the system's availability target."
- **Error budgets end the dev/ops conflict.** SRE's goal is no longer "zero outages". Both teams spend the budget to get as much feature velocity as they can, and outages become "an expected part of the process of innovation."
- **Blame-free postmortem culture** (*Ensuring a Durable Focus on Engineering*).
- **Software interprets monitoring, humans act** (*Monitoring*): "Monitoring should never require a human to interpret any part of the alerting domain."
- **SRE owns capacity planning, so SRE must also own provisioning**, and through that it is involved in utilization and efficiency work.
- **DevOps vs SRE** (sidebar): DevOps can be seen as a generalization of core SRE principles, or SRE as "a specific implementation of DevOps with some idiosyncratic extensions."

## Concrete numbers, thresholds and formulas (as written)

| Item | Value | Section |
|---|---|---|
| SRE hiring mix | 50–60% hired through the standard Google SWE process; 40–50% "very close" to SWE qualifications (85–99% of the skill set) plus rare skills, most often UNIX system internals and networking (Layer 1 to Layer 3) | Google's Approach |
| Ops work cap | **50% cap on aggregate "ops" work** (tickets, on-call, manual tasks). This is an *upper bound*. The remaining 50% must go to development | Google's Approach; Durable Focus |
| On-call event load | **Maximum of two events per 8–12-hour on-call shift**, on average. Consistently **fewer than one** per shift wastes the engineer's time | Durable Focus |
| Error budget | **error budget = 1 − availability target**. For example, 99.99% available means 0.01% unavailable, and that 0.01% is the budget | Pursuing Maximum Change Velocity |
| Reliability | A function of **MTTF and MTTR**. MTTR is the most relevant metric for judging emergency response | Emergency Response |
| Playbooks | Roughly a **3x improvement in MTTR** compared with "winging it" | Emergency Response |
| Change-caused outages | **Roughly 70%** of outages come from changes in a live system | Change Management |
| SRE founding | Started in 2003 with a "Production Team" of seven engineers | Google's Approach |
| DevOps term | Appeared in late 2008. The chapter was written in early 2016 | DevOps or SRE? |

## Recommended practices and procedures

- **Measure how SRE time is spent.** When ops work goes over 50%, send the excess to the product development teams: reassign bugs and tickets to dev managers and put developers back into on-call pager rotations. Another remedy is adding staff without adding ops responsibilities. Stop redirecting once the load is at or below 50%. This "safety valve" also gives developers feedback that pushes them to build systems that need no manual intervention.
- **Write postmortems for all significant incidents, whether or not they paged.** Incidents that did not page are "even more valuable" because they likely point to monitoring gaps. A postmortem should establish what happened, find *all* root causes, and assign actions.
- **Use error budgets for release decisions.** Phased rollouts and 1% experiments "free up" budget so launches can go faster. Example consequence: release freezes "for the remainder of the quarter once an error budget is depleted." This needs management backing.
- **Questions for setting a target:** (1) What availability will users be happy with, given how they use the product? (2) What alternatives do dissatisfied users have? (3) How does usage change at different availability levels?
- **Emergency response:** keep on-call playbooks and run exercises such as the "Wheel of Misfortune" (disaster role playing).
- **Change management:** use automation to (1) roll out progressively, (2) detect problems quickly and accurately, and (3) roll back safely. Taking humans out of the loop avoids fatigue, familiarity/contempt, and inattention.
- **Provisioning** is riskier than load shifting, which "is often done multiple times per hour". Do it quickly, only when needed (capacity is expensive), and validate that the new capacity works correctly.
- **Performance:** provision to a capacity target "at a specific response speed". A slowdown is a loss of capacity, and a system that stops serving is "infinite slowness."

## Anti-patterns Google warns against

- **The dev/ops split** (*The Sysadmin Approach*). It has direct costs: team size scales with load. It also has indirect costs, often larger: different vocabulary, risk assumptions, and incentives, which can erode "trust and respect". The chapter calls this "a pathology."
- **"Trench warfare" over launches.** Ops builds launch gates, such as reviews that check for every problem that ever caused an outage. Dev works around them with "flag flips," "incremental updates," "cherrypicks," and sharding products so less gets reviewed.
- **Email alerting on thresholds.** It is "fundamentally flawed" when a human has to read an email and decide whether to act.
- **Pager overload.** More than two events per shift on a regular basis prevents thorough investigation and learning. Pager fatigue "won't improve with scale."
- **"Winging it"** and relying on the hero jack-of-all-trades on-call engineer instead of a practiced engineer with a playbook.
- **Targeting 100% reliability**, or "zero outages" as a goal.
- **Hiring more people to repeat the same tasks** as the service grows.
- **Not having capacity in place by the time it is needed.**

## Templates and checklists

**The three valid kinds of monitoring output** (*Monitoring*):
1. **Alerts**: a human must act *immediately*, because something is happening or about to happen.
2. **Tickets**: a human must act, but not immediately. "If a human takes action in a few days, no damage will result."
3. **Logging**: no one needs to look at it. It is kept for diagnostic or forensic use and read only when something else prompts it.

**Mandatory capacity planning steps** (*Demand Forecasting and Capacity Planning*):
1. An accurate organic demand forecast that extends beyond the lead time needed to acquire capacity.
2. Accurate incorporation of inorganic demand (launches, marketing campaigns, business-driven changes).
3. Regular load testing to correlate raw capacity (servers, disks) with service capacity.

**SRE team responsibilities** (*Tenets of SRE*): availability, latency, performance, efficiency, change management, monitoring, emergency response, and capacity planning.

**Efficiency model**: resource use is a function of demand (load), capacity, and software efficiency.

## Short verbatim quotes

- "SRE is what happens when you ask a software engineer to design an operations team."
- "the team tasked with managing a service needs to code or it will drown."
- "we want systems that are automatic, not just automated."
- "100% is the wrong reliability target for basically everything"
- "the error budget is one minus the availability target."
- "Monitoring should never require a human to interpret any part of the alerting domain."
- "Humans add latency."
