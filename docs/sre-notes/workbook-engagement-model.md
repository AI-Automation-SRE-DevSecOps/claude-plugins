# SRE Workbook, Chapter 18: SRE Engagement Model

Source: https://sre.google/workbook/engagement-model/ (Wildpaner, Sheerin, Rogers, Sanagavarapu (NYT), with Hilton and Nukala). The chapter builds on Ch. 32 of the first SRE book (PRRs, early engagement, continuous improvement).

Extraction note: the plain-text extraction dropped the SRE-to-developer ratio because the HTML contains a literal `<`. The raw HTML reads: "Google generally maintains an SRE-to-developer ratio of < 10%. Therefore, one SRE team commonly works with multiple developer teams in their product area (PA)."

## Core principles
- SRE aims to "maximize the engineering velocity of developer teams while keeping products reliable." An SRE team can't cover every service, so it has to choose where to focus. It makes that choice together with the developer and product teams.
- An engagement is about understanding what the developer and product teams are trying to achieve, not only about the service itself. A "team" can start as a single person.
- SRE adds value across the whole service lifecycle, often "well before any SRE picks up the pager" (The Service Lifecycle).
- Each team keeps its own focus and also explicitly supports the other team's goals (Aligning Goals).
- Engagements are not necessarily permanent. SRE exists to do impactful engineering work (Ending the Relationship).

## Concrete numbers and thresholds (exact)
- **SRE-to-developer ratio:** "< 10%" (Supporting Multiple Services with a Single SRE Team).
- **Lifecycle phases (Figure 18-1):** 1 Architecture and Design, 2 Active Development, 3 Limited Availability, 4 General Availability, 5 Deprecation, 6 Abandoned, 7 Unsupported.
- **SLOs:** "We strongly recommend defining SLOs before general availability (GA)" (Phase 3).
- **Error budget:** an "agreed-upon **quarterly** error budget that determines release velocity and other safety parameters" (Setting Ground Rules).
- **Service reviews:** SRE and developer leads meet face-to-face "at least once a year" and share roadmaps for "the next 12–18 months."
- **Org-wide summit:** the entire distributed team gathers "every 12–18 months."
- **Rotating site responsibilities:** rotate "regularly (e.g., yearly)," and don't lock a responsibility to one location "for multiple years."
- **State-of-production talk:** quarterly, given by SRE to product development leadership.
- **NYT feedback loop:** for example, a "biweekly retrospective or check-in with team managers."
- **Data pipeline case:** the pivot happened 3 years before turndown, after about a decade of cooperation. The old and new systems were expected to overlap for 18–24 months.
- **Minimum footprint:** keep at least a two-location arrangement, even if the locations are just separate buildings.

## Recommended practices by phase (The Service Lifecycle)
- **Phase 1, Architecture and Design:** SRE writes best practices (for example, resilience to SPOFs) and documents the dos and don'ts of infrastructure. It consults early and uses targeted prototypes to test assumptions. It may also join the dev team or codesign part of the service. Architectural mistakes get harder to fix later.
- **Phase 2, Active Development:** productionizing. This covers capacity planning, redundancy, spike and overload handling, load balancing, monitoring, alerting and performance tuning.
- **Phase 3, Limited Availability (Beta):** define SLOs, build a capacity model, acquire resources, and automate turnups and resizing. Write alerts that match the upcoming SLOs. Share incident and operational work between devs and SREs. The product can still be withdrawn if it can't meet its reliability target.
- **Phase 4, GA:** the service has passed the PRR. SRE does most of the operational work, but devs keep "a small part," for example by permanently putting one developer in the on-call rotation.
- **Phase 5, Deprecation:** SRE runs the old system mostly without the devs and supports the migration. SRE is "effectively supporting two full systems," so adjust headcount to match.
- **Phase 6, Abandoned:** devs usually take operational support back, and SRE helps on a best-effort basis. For internal services, management is handed to the remaining users.
- **Phase 7, Unsupported:** SRE removes references to the service from production configs and docs.

## Ground rules and relationship mechanics
**Setting Ground Rules.** Every Google SRE team has two goals:
- Short term: a stable, available system that scales with demand.
- Long term: optimize operations "to a level where ongoing human work is no longer needed."

The principles of cooperation are:
1. Definitions of operational work, with a **hard limit** on it.
2. An agreed-upon, measured SLO that prioritizes engineering work for both teams. You can start without one, but you will have to come back to it later (see the Waze case study).
3. An agreed-upon quarterly error budget.
4. Developer involvement in daily operations.

**Planning and Executing.** Publish yearly roadmaps with developer leadership and derive quarterly goals from them. In a stable environment, having no roadmap may mean the SRE team should merge, hand work back, expand scope or dissolve.

**Reassessing When Ground Rules Start to Slip.** There are four escalating responses:
1. Named engineers drop lower-priority work to fix the regression.
2. A "reliability hackathon" runs, and normal priorities continue outside it.
3. A feature freeze, where most of both teams work on the regression.
4. An "all hands on deck" response, when technical leadership judges reliability to be at acute risk.

**Adjusting Priorities by SLO and Error Budget.**
- If the SLO is at risk or the budget is exhausted, both teams treat recovery as high priority. They can use tactical fixes (overprovisioning) and strategic ones (optimizations, caching, graceful degradation).
- If the service has ample budget left, spend it on feature velocity rather than "overproportional efforts on service improvements."

**Relationship habits.** Meet counterparts regularly, including upstream and infrastructure SRE teams, so escalations go smoothly. Give a quarterly "state of production" talk, and have devs give a "state of the product" talk in return. Run a stop/keep/start retrospective; it needs active facilitation and is often rated the most useful session.

**Handling Mistakes Appropriately.** Be blameless. Sleep on it before follow-up conversations, and use video across locations. Meet in person rather than arguing through code reviews or docs. Be positive and call out what was good. Understand that teams communicate differently.

## Scaling
- One SRE team can cover many services when they are part of **a single product**, built on **similar tech stacks**, and built by **the same developer team or a small number of related ones**.
- The Google SRE organization shadows the developer PA hierarchy. To group SRE teams, use either product or technology stack. To prevent churn during reorgs, Google recommends organizing **by technology** rather than by the developer PA reporting structure.
- Create, shard, merge or dissolve teams based on service needs and load. Every team should have a clear charter. When a team owns too many services, Google prefers to **shard the existing team** rather than build a new one from scratch. Only restructure when necessary.
- Singleton teams are "generally less effective and more vulnerable" to reorgs.

## Anti-patterns
- Skipping the SLO at the start of the relationship.
- Locking a responsibility to one site for years, which breeds "us versus them," blocks knowledge sharing and creates a business continuity risk.
- Singleton teams.
- Restructuring teams without need.
- Resolving disputes only through code reviews or docs.
- Having follow-up conversations while tired or emotional.
- Using SREs as traditional ops: NYT says SREs "do not support manual work such as running a job for deployment."
- One-off scripts instead of company-wide tooling (NYT).
- Keeping toil-heavy engagements where SRE's value proposition is gone.
- Letting communication gaps grow between an SRE team on the old system and a dev team on the new one (Case Study 2).

## Templates and checklists
**NYT shared goals (Shared Goals: SRE Engagements at the New York Times).**
- Engagements are full-time (the SRE is embedded in the product team) or part-time (brief, constrained projects).
- Application owners, not SREs, make the changes to the application.
- Fixes from ARRs and PRRs are prioritized jointly.
- Goal pattern: (1) define the scope, for example "In the next quarter, I want SRE to work with the dev team to stabilize the app…"; (2) state the end-result success story explicitly, for example "the product development team can handle our service outages in Google Kubernetes Engine without escalation."
- Roadmap: review architecture → define shared goals → kickoff and planning → development cycles → retrospectives → PRRs → development cycles → plan and execute launches.
- Agree on a feedback method and how often it happens. Plan for disengagement if the engagement isn't working.
- Measuring Impact: do a point-in-time maturity-matrix assessment before the engagement. It scores areas such as observability, capacity planning, change management and incident response. Repeat the assessment afterwards, then celebrate.

**Reasons to hand a service back (Ending the Relationship):**
- The service is optimized enough that SRE is no longer needed.
- Its importance has diminished.
- It is nearing end of life.

**Case studies.**
- **Ares:** CAT built an in-house infrastructure team with production expertise and an abstraction layer. SRE then disengaged from CAT and focused on the underlying infrastructure, and velocity went up.
- **Data Analysis Pipeline:** two SRE teams were decommissioned and the devs took full control. In a healthier relationship, SRE would have handed the system back only briefly and then taken it back once it was stable.

## Short quotes
- "We will support you in releasing as quickly as is safe," where "safe" generally implies staying within error budget.
- "Definitions of (and a hard limit on) operational work."
- "The absence of a roadmap may be a signal that the SRE team can merge with another team…"
- "We've found that singleton teams are generally less effective and more vulnerable to the effects of reorgs outside the team."
- "Investing in aligning team goals and understanding each other's objectives is as important as defending SLOs."
