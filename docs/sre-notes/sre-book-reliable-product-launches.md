# Chapter 27 — Reliable Product Launches at Scale (Google SRE Book)

Source: https://sre.google/sre-book/reliable-product-launches/ — Rhandeev Singh and Sebastian Kirsch with Vivek Rau; ed. Betsy Beyer.

## Core principles

- **Definition of a launch:** "any new code that introduces an externally visible change to an application." Under this definition Google sometimes does **up to 70 launches per week** (intro).
- A high launch rate is both the reason for, and the opportunity to build, a streamlined launch process. A company launching every three years doesn't need one (intro).
- **Launch Coordination Engineering (LCE)** is a dedicated consulting team inside SRE. It audits services against reliability standards, acts as liaison between teams, drives technical momentum, **acts as gatekeeper / signs off on "safe" launches**, and educates developers (§ Launch Coordination Engineering).
- LCEs meet the SRE technical bar plus leadership skills, and bring breadth, a cross-functional view, and objectivity. As SREs they are incentivized to put reliability first (§ The Role of the LCE).
- **Criteria for a good launch process** (§ Setting Up a Launch Process): Lightweight, Robust, Thorough, Scalable, Adaptable. Some of these conflict (lightweight vs. thorough). Three tactics balance them: **Simplicity** ("Get the basics right. Don't plan for every eventuality."), **a high-touch approach** (experienced engineers tailor the process to each launch), and **fast common paths** for recurring launch classes, e.g. launching in a new country.
- Load testing is **required for most launches**, because overload behavior is hard to predict from first principles (§ Overload Behavior and Load Tests).

## Concrete numbers / thresholds (exact)

| Item | Value | Section |
|---|---|---|
| Launch rate | up to 70 launches per week | Intro |
| Keyhole on Christmas Eve 2011 | 25 times normal peak; upward of one million requests per second (normal: "up to several thousand" images/s) | Intro |
| Launch spikes vs. estimates | up to 15 times higher than initially estimated | Capacity Planning |
| Redundancy rule | if 3 replicated deployments serve 100% of peak, maintain 4 or 5 (1–2 redundant) | Capacity Planning |
| Feature-flag ramp | "a larger but limited group of users, usually between 1 and 10 percent"; frameworks ramp 0% to 100% | Feature Flag Frameworks |
| Sync-interval example | sync every 60 s instead of 600 s = 10 times the load | Abusive Client Behavior |
| Retry backoff example | 1 s, 2 s, 4 s… must be jittered | Abusive Client Behavior |
| Low-risk launch definition | no new server executables and a traffic increase under 10% | Evolution of the LCE Checklist |
| Low-risk share | by 2008, 30% of reviews | Evolution of the LCE Checklist |
| Throughput | one LCE ran 350 launches in 3.5 years; team averaged five engineers, so over 1,500 launches in 3.5 years | Evolution of the LCE Checklist |
| LCE ramp-up | about six months of training for a new hire | Evolution of the LCE Checklist |
| Full-time LCE team founded | 2004 | Development of LCE |
| Process maturity | honed over more than 10 years | Setting Up a Launch Process |
| Checklist full review | once or twice a year | The Launch Checklist |
| Ops-work cap | operational work below a maximum of 50% | Problems LCE Didn't Solve |
| Rearchitecture trigger | usage up by more than two orders of magnitude | Problems LCE Didn't Solve |

## Recommended practices & procedures

- **Checklist curation** (§ The Launch Checklist): each item pairs a *Question* with an *Action item* and a pointer to resources. Two admission rules: every question's importance "must be substantiated, ideally by a previous launch disaster," and every instruction must be "concrete, practical, and reasonable." At one point, adding a question needed **VP approval**. Make small updates continuously and do a full review once or twice a year.
- **Drive convergence** (§ Driving Convergence and Simplification): use the checklist to push teams onto shared, hardened infrastructure for rate limiting/quotas, data pushes, and binary releases. Long sections can then shrink to one line: "Implement rate limiting using system X." LCEs also simplify painful approval processes (for example, automatic approvals for common cases) and escalate pain points to infrastructure owners.
- **Launching the unexpected:** for a new domain (e.g. Android, where client-side bugs can't be fixed quickly by a server push), build the checklist around broad themes such as reliability, failure modes, and processes. Bring in domain experts and keep each question's *intent* in mind.
- **Gradual and staged rollouts:** a few machines in one datacenter, observe; then all of one datacenter, observe; then global. These early stages are "canaries." Tooling rolls back automatically if a change fails the validation period. The same idea applies to percentage-based Android app rollouts, invite systems, and rate-limited signups.
- **Feature flag frameworks** must: roll out many changes in parallel; ramp to 1–10% of users; route by user/session/object/location; handle new-code failure by design without affecting users; revert each change independently and immediately; and measure UX impact. There are two classes: UI-focused (e.g. an HTTP payload rewriter scoped by cookie hash mod range, whitelists/blacklists) and server-side/business-logic. Stateful services scope by user ID or entity ID and proxy/reroute instead of rewriting.
- **Client behavior:** use exponential backoff and jitter. Retry network errors but usually not 4xx errors. Randomize scheduled times instead of using a fixed time like 2 a.m. Control the client from the server through a downloaded config (sync/retry rates, feature toggles), and ship **dormant functionality** so a launch can be aborted by switching it off.
- **Processes:** document all manual processes so any team member can run them in an emergency (humans are single points of failure too). Automate build and release.
- **Development:** check all code *and config* into version control. Develop on mainline, and cut each release on a new release branch.
- **Rollout planning:** write a launch plan with an owner per step, identify the risk in each step, and prepare contingencies, e.g. a backup keynote slide saying "We will be launching this feature over the next days."
- **Infrastructure churn:** adopt a churn-reduction policy. Infrastructure teams may not release backward-incompatible features until they automate their clients' migration.

## Anti-patterns warned against

- Processes so burdensome that engineers sidestep them, especially in crunch mode.
- Letting the checklist grow without limit ("near-infinite number of questions").
- Applying checklist items mechanically to a novel product without regard to their intent.
- Re-implementing common infrastructure (e.g. rate limiting) instead of reusing it.
- "Push-button" global launches; assuming a config that works in test will roll out cleanly to live.
- Synchronized clients (thundering herd), un-jittered retries, retrying 4xx errors, and overly aggressive sync intervals.
- Assuming CPU scales linearly with load. Examples of collapse: debug logging that costs more than normal handling, and JVM "GC thrashing."
- Overly cautious, change-averse SREs; bureaucracy creep (by 2009 launching a small service "had become a legend").

## Checklist themes (template; full list in Appendix E)

Architecture and Dependencies (request flow; isolate user-facing from non-user-facing traffic; one page view can become many requests) · Integration (DNS, load balancers, monitoring) · Capacity Planning (promotion tie-ins, expected traffic and growth, resources obtained) · Failure Modes (SPOFs, dependency unavailability at startup/runtime, request deadlines, load shedding) · Client Behavior (auto-save/heartbeat, backoff, jitter) · Processes and Automation · Development Process · External Dependencies (third parties, partners, hard deadlines) · Rollout Planning.

"Launch Reviews" happened days to weeks before launch and were later formalized as "Production Reviews."

## Verbatim quotes

- "Google defines a launch as any new code that introduces an externally visible change to an application."
- "Every question's importance must be substantiated, ideally by a previous launch disaster."
- "Simplicity: Get the basics right. Don't plan for every eventuality."
- "If the change doesn't pass the validation period, it's automatically rolled back."
- "it is very hard to predict from first principles how a service will react to overload."
- "The LCE team was Google's solution to the problem of achieving safety without impeding change."
