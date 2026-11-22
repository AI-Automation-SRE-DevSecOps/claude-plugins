# Chapter 18 — Software Engineering in SRE (Google SRE Book)

Source: https://sre.google/sre-book/software-engineering-in-sre/
Authors: Dave Helstroom and Trisha Weir, with Evan Leonard and Kurt Delimon; edited by Kavita Guliani.

Structure: why SRE should build software, then a case study of **Auxon** (intent-based capacity planning), then guidance on fostering software engineering inside SRE.

## Core principles

- **SRE tools are products, not hacks** (intro). SRE-built tools are "full-fledged software engineering projects, distinct from one-off solutions and quick hacks." They need a product mindset that covers internal customers and a roadmap.
- **Sublinear team growth** (*Why Is Software Engineering Within SRE Important?*). On purpose, SRE-supported services grow faster than the SRE org. The guiding principle is "team size should not scale directly with service growth." Keeping team growth linear while services grow exponentially takes "perpetual automation work."
- **SREs are well placed to build these tools.** They have deep production knowledge (scalability, graceful degradation, how to interface with other infrastructure). They are embedded in the problem. Their users are fellow SREs, which gives "frank and high-signal" feedback.
- **The SRE org benefits too.** Project work offers career development and keeps coding skills fresh. It balances interrupts and on-call work, and it helps hire and keep a diverse mix of engineers. Google staffs SRE teams with a mix of software-development and systems-engineering backgrounds.
- **Specify the requirements, not the implementation** (*Our Solution: Intent-Based Capacity Planning*). Encode a service's dependencies and parameters (its *intent*) in a program, then autogenerate the allocation plan. When demand, supply or requirements change, regenerate the plan.
- **Launch and iterate** (*Approximation*). Don't chase a perfect, pure solution, especially when the problem's bounds are unclear.
- **SREs who develop must stay SREs** (*Staffing and Development Time*). They should keep working as SREs and not become full-time developers inside the SRE org, so that they remain "both the creator and the customer."

## Concrete numbers, thresholds, formulas

- **Traditional capacity-planning cycle** (*Traditional Capacity Planning*), 4 steps:
  1. Collect demand forecasts. These cover "anywhere from several quarters to years."
  2. Devise build and allocation plans.
  3. Review and sign off on the plan.
  4. Deploy and configure resources.
- **Intent abstraction ladder** (*Intent-Based Capacity Planning*):
  1. "I want 50 cores in clusters X, Y, and Z for service Foo."
  2. "I want a 50-core footprint in any 3 clusters in geographic region YYY for service Foo."
  3. "I want to meet service Foo's demand in each geographic region, and have N + 2 redundancy."
  4. "I want to run service Foo at 5 nines of reliability."
  - Services get the best wins when they cross to **step 3**. "Particularly sophisticated services may aim for step 4." Ideally all levels are supported together.
- **Example constraints.** Bar must be "within 30 milliseconds of network latency of Foo." In Auxon: "My service must be N + 2 per continent". "The frontend servers must be no more than 50 ms away from the backend servers."
- **Bin packing is NP-hard.** Even so, "today's algorithms can solve to a known optimal solution." Manual bin packing has "no known bounds on an optimal solution."
- **Auxon facts.** It was built by "a small group of software engineers and a technical program manager within SRE over the course of two years." It plans "many millions of dollars of machine resources." Intent is represented internally as "a giant mixed-integer or linear program." The solver runs in parallel on "hundreds or even thousands of machines."
- **Linear-program mapping** (*Introduction to Auxon*, Figure 18-1). Resource Supply is the **upper bound**. Resource Pricing feeds the **objective** to minimize, which is cost.
- **Release cadence** (*Getting There*). A "six-month rhythm of product update releases." Some teams later moved to **push-on-green**.

## Auxon components (Figure 18-1)

| Component | Role |
|---|---|
| Performance Data | How a service scales: units of dependency Z per unit of demand X in cluster Y. Comes from load tests or past performance. |
| Per-Service Demand Forecast Data | Forecasted usage trends (e.g., QPS by continent). Some services, like Colossus, have no forecast of their own: their demand comes from the services that depend on them. |
| Resource Supply | Future availability of base resources. This is the LP upper bound. |
| Resource Pricing | Cost of base resources (e.g., space and power by facility). This is the LP objective. |
| Intent Config | A human-readable layer that defines services and how they relate. It wires all the components together. |
| Auxon Configuration Language Engine | Turns the config into a protocol buffer request and applies light sanity checks. |
| Auxon Solver | Formulates and solves the MILP/LP. Also handles scheduling, a worker pool and decision trees. |
| Allocation Plan | The output: which resources go to which services, where. It also reports **requirements that could not be satisfied**. |

**Precursors to intent:** Dependencies (nested and shared), Performance metrics ("the glue between dependencies", derived from load testing and usage monitoring), and Prioritization (made "transparently, openly, and consistently").

## Recommended practices

- **Approximate first, behind an abstraction.** The "Stupid Solver" was a heuristic placeholder behind an abstracted solver interface. Later it was swapped for a real LP solver.
- **Use fuzzy requirements as a reason to be general and modular.** Auxon made the Allocation Plan "universally useful," so it was agnostic to automation, forecasting and performance-data tools. Machine-performance models sat behind a single swappable interface.
- **Agnosticism** (*Designing at the right level*): "come as you are; we'll work with what you've got." Avoid over-customizing for one or two big users.
- **Driving adoption** (*Raising Awareness*) needs three things: a consistent and coherent approach, user advocacy, and sponsorship from senior engineers and management. Provide documentation.
- **Set expectations.** Separate aspirational goals from the Minimum Viable Product. Ship small, incremental releases. Pair a long-term roadmap with short-term fixes.
- **Target the right customers first.** Start with teams that have *no existing process*. Publish a before/after case study to quantify the benefit.
- **White-glove support for early adopters.** It also addresses fears like "someone's job will be replaced by a shell script." Early adopters then act as local experts for other teams.
- **Team composition.** Seed the team with fast-learning generalists plus broadly experienced engineers. Engage specialists (Auxon used Operations Research and Quantitative Analysis) once the project is "successfully off the ground and demonstrably successful."
- **Dedicated, uninterrupted project time** "must be aggressively defended."
- **Partner with SWEs, TPMs or PMs** to build a product culture, as a counter to the attitude "I have a design doc; why do we need requirements?"

## Anti-patterns

- Spreadsheet capacity planning. It scales poorly, has limited error checking and goes stale. Capacity requests like "X cores in cluster Y" lose their reasons and degrees of freedom.
- Defining success as **100% adoption**. Closing the long-tail last mile has diminishing returns.
- Poor candidate projects: ones that touch many moving parts at once, or need an all-or-nothing design that blocks iteration.
- Projects that are **overly specific** to one service, or frameworks that are **overly generic** and fit no use case well. Also projects with grand scope, abstract goals and no concrete use cases.
- Ad hoc, narrow, single-purpose tools that can't be shared as the org grows, which leads to duplicated effort.
- Promising too much too soon. Cutting corners on quality.

## Checklists

**Good candidate project** (*Fostering Software Engineering in SRE*):
- [ ] Engineers with firsthand domain experience want to work on it.
- [ ] The users are highly technical, so they give high-signal bug reports.
- [ ] It brings a noticeable benefit: less toil, better infrastructure, or a simpler complex process.
- [ ] It fits the org's objectives and has been socialized across the org, to avoid overlapping efforts.

**Getting There** guidelines (*Getting There*):
1. Create and communicate a clear message. You've passed the first hurdle when SREs ask *how* the strategy will work, not *if*.
2. Evaluate your org's capabilities. Do you have a PM or customer advocate? Agile skills? Borrow from product teams.
3. Launch and iterate. Start with uncontroversial targets that have no existing solutions.
4. Don't lower your standards:
   - [ ] Would you onboard this product if a separate dev team had built it?
   - [ ] Is there code review?
   - [ ] Is there end-to-end or integration testing?
   - [ ] Has another SRE team done a production-readiness review?

**Paths for a side project:** stay grassroots, become a formal project, or gain executive sponsorship and become fully staffed.

## Short verbatim quotes

- "team size should not scale directly with service growth."
- "Specify the requirements, not the implementation."
- "SREs are busy, and if your solution is too difficult or confusing, they will write their own solution."
- "It takes a long time to build credibility for your software development efforts, but only a short time to lose credibility due to a misstep."
- "Don't wait for the perfect design; rather, keep the overall vision in mind while moving ahead with design and development."

Other: a layer-3 load balancer built by Google SREs became Google Cloud Load Balancer [Eis16].
