# Chapter 32 — The Evolving SRE Engagement Model (SRE Book)

Source: https://sre.google/sre-book/evolving-sre-engagement-model/ (Acacio Cruz and Ashish Bhambhani; edited by Betsy Beyer and Tim Harvey)

## Core principles

- **Onboarding** is the process of evaluating a service, checking that it merits SRE support, negotiating fixes for any gaps that block support, and then starting SRE support (*SRE Engagement: What, How, and Why*).
- **Earlier is cheaper.** Bugs cost less to fix the earlier they are found, and the same holds for SRE involvement. Engaging SRE at the design stage cuts onboarding time and makes the service more reliable "out of the gate."
- **The best option is a platform.** Give product development SRE-validated infrastructure to build on, so developers can "focus on innovation at the application layer."
- **What SRE means by "production"** (*The SRE Engagement Model*) has six aspects: system architecture and interservice dependencies; instrumentation, metrics, and monitoring; emergency response; capacity planning; change management; and performance (availability, latency, and efficiency).
- **A PRR is a prerequisite.** An SRE team accepts production responsibility for a service only after a Production Readiness Review.
- **The chapter names three engagement models:** the Simple PRR Model, the Early Engagement Model, and Frameworks and SRE Platform. "All three of the engagement models described are still practiced within Google" (*Conclusion*). The chapter's intro calls the middle one the "Extended Engagement Model"; elsewhere it is "Early Engagement."
- **Not every service gets SRE** (*Alternative Support*). Many services don't need high reliability, and by design more teams request SRE than SRE can staff. The fallbacks are **documentation** (the Production Guide) and **consultation** (for example, the Launch Coordination Engineering team).

## Concrete numbers and thresholds (as written)

| Item | Value |
|---|---|
| Launch consultation | "one or two SREs spending a few hours" on a high-level review of design and implementation |
| PRR team (Engagement phase) | "Usually one to three SREs are selected or self-nominated" |
| Onboarding under the Simple PRR / Early models | "required two or three SREs and typically lasted two or three quarters"; PRR lead times "quarters away" |
| Lead time of the old PRR model | "a lead time of months" (described as too slow for microservices) |
| Onboarding with frameworks | "usually accomplished by a single SRE during one quarter" |
| Deployment on the framework platform | an idea reaches "fully deployed SRE-level production quality in a matter of days!" |
| Framework languages | one framework per supported environment: "Java, C++, Go" |
| Original support options | only two: "either full SRE support, or approximately no SRE engagement" |

## Recommended practices and procedures

**Simple PRR Model phases** (*Production Readiness Reviews: Simple PRR Model*). This model is usually used for a service that has already launched. It runs only after SRE has judged the service important enough, confirmed SRE teams are available, and agreed staffing levels with the development team.

1. **Engagement.** SRE leadership picks the team. One to three SREs agree with the developers on an **SLO/SLA**, plans for disruptive design changes, and the planning and training schedule.
2. **Analysis.** This is the first large block of work. Reviewers measure the service's maturity on each production axis, using a service-specific **PRR checklist** built from domain expertise, similar systems, and the Production Guide. They also review recent incidents, postmortems, and the follow-up items from those incidents.
3. **Improvements and Refactoring.** Reviewers prioritize fixes by their importance to reliability, then negotiate an execution plan with the developers. Both teams do the work. This phase "varies the most in duration and amount of effort."
4. **Training.** The PRR reviewers train the whole SRE team, usually with the developers: design overviews, deep dives on request flows, the production setup, and hands-on operations exercises.
5. **Onboarding.** Responsibilities move over progressively (operations, change management, access rights). The development team must stay available to back up and advise SRE for a period afterward.
6. **Continuous Improvement.** SRE keeps up reliability standards as the service changes, and feeds lessons back to the developers and into the Production Guide.

**PRR objectives:** confirm the service meets production-setup and operational-readiness standards and that its owners are ready to work with SRE; improve reliability and minimize "the number and severity of incidents that might be expected."

**Early Engagement candidates** (*Candidates for Early Engagement*):
- significant new functionality that will be part of a system SRE already manages;
- a significant rewrite of, or alternative to, an existing system for the same use cases;
- a development team that asked SRE for advice or for takeover at launch.

In this model SRE takes part from the Design phase onward and can take over "any time during or after the Build phase."

**Early Engagement practices by phase:**
- **Design.** SRE is part of trade-off decisions, which minimizes later disputes over design choices.
- **Build.** SRE recommends existing libraries and builds in operational and emergency controls.
- **Launch.** Use a **dark launch**: copy part of existing users' traffic to the new service, then throw away its responses.
- **Disengaging** is acceptable. If a service turns out not to need SRE, that is "a positive outcome." SRE effort spent on projects that miss their usage projections is part of normal business risk.

**Frameworks principles** (*Toward a Structural Solution: Frameworks*): codified best practices; reusable solutions; a common production platform with a common control surface; easier automation and smarter systems. Typical framework modules cover instrumentation and metrics, request logging, and traffic and load-management control systems. A framework provides standard monitoring dimensions, a standard request-debug log format, a standard load-shedding config format, and a semantically consistent definition of server capacity and "overload." Implementations in different languages can't share code, but they should expose the same API, behavior, configuration, and controls.

**Shared-responsibility model.** SRE supports the platform: load shedding, overload, automation, traffic management, logging, and monitoring. Developers take on-call for functional issues, meaning bugs in application code. Platform teams are staffed "based upon the need to maintain the platform rather than upon service count" (footnote 153).

## Anti-patterns and limitations Google calls out

- **Engaging late.** The main limitation of the Simple PRR Model is that the service is already "launched and serving at scale" and SRE arrives very late.
- **SRE working as an external unit.** SREs "should essentially work as a part of the development team." Their work must be highly visible and reviewed by the developers.
- **Per-service reimplementation of production features.** The canonical example is functionally similar logging frameworks built repeatedly in the same language, which the chapter calls "a waste of engineering effort."
- **Fixes that don't replicate.** Recurring problems such as overload and data hot-spotting had no easy way to spread fixes across services. SRE code stayed local to each service.
- **Linear scaling.** PRR effort grew with the number of services, which led to "serialization of service takeovers and strict service prioritization." With microservices, every service carries "a base fixed operational cost."
- **Relying on consultation for large or heavily depended-on services.** Consultation is "necessarily broad in scope." It is not enough for services that have grown by orders of magnitude or that many clients now depend on.

## Template: example PRR Analysis checklist items (verbatim)

- Do updates to the service impact an unreasonably large percentage of the system at once?
- Does the service connect to the appropriate serving instance of its dependencies? (End-user requests should not depend on a system designed for batch processing.)
- Does the service request a sufficiently high network quality-of-service when talking to a critical remote service?
- Does the service report errors to central logging systems for analysis? Does it report all exceptional conditions that degrade responses or cause failures for end users?
- Are all user-visible request failures well instrumented and monitored, with suitable alerting configured?
- (The checklist can also include the team's "gold standard," for example configs refactored to work with SRE config tooling.)

**Worked example (Shakespeare).** The PRR found that the dashboards did not fully cover the SLO metrics. Once the filed issues were fixed, SRE took the pager, with two developers staying in the on-call rotation and joining the weekly on-call meeting.

## Short quotes

- "A PRR is considered a prerequisite for an SRE team to accept responsibility for managing the production aspects of a service."
- "The best production incidents are those that never happen!"
- "SREs should essentially work as a part of the development team, rather than an external unit."
- "This practice effectively breaks the SRE staffing barrier."
- "Because each service has a base fixed operational cost, even simple services demand more staffing."
