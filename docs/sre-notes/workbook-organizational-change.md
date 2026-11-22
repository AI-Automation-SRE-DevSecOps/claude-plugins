# SRE Workbook, Chapter 21: Organizational Change Management in SRE

Source: https://sre.google/workbook/organizational-change/
Authors: Alex Bramley, Ben Lutch, Michelle Duffy, Nir Tarcic with Betsy Beyer

## Scope and definition (intro)
- "Change management" here means **organizational** change management: preparing and supporting individuals, teams, and business units through organizational change. It explicitly does **not** mean project-management *change control* (change review, versioning).
- Ben Treynor Sloss (SRE Book intro) lists organizational change management as a core SRE responsibility.
- Error budgets are the primary mechanism for balancing change risk against reliability ("SRE Embraces Change").
- Google SRE has **not** formally applied any of these frameworks' processes; it uses them as a lens to refine its approach.

## Core principles
- No single model fits every situation; Google SRE has not standardized on one ("How These Theories Apply to SRE").
- Change is cyclical: removing one bottleneck exposes the next (Kotter insight, Waze deployment cycle).
- Incremental change is easier to manage, but still needs a broad master plan ("Lessons Learned", Waze).
- Grassroots change needs SRE + developer collaboration *and* executive support; ad hoc groups should evolve into formal, structured cooperation.
- For common tooling, the first measure of success is **adoption** (Case Study 2, "Lessons Learned").
- Most grassroots changes benefit from structured coordination and planning as they mature (Conclusion).

## The models, and Google's verdict on each ("Introduction to Change Management")
| Model | What it is | Applicability to SRE per chapter |
|---|---|---|
| Lewin's three-stage (unfreeze–change–freeze) | 1) persuade group change is needed, 2) execute, 3) institutionalize; group is the primary dynamic instrument | Most useful for macro-level planning |
| McKinsey 7-S | Structure, strategy, systems (business); shared values, skills, style, staff (people) | Useful for moving from traditional sysadmin to SRE |
| Kotter's Eight-Step Process (*Leading Change*, 1996) | Urgency → coalition → vision → volunteers → remove barriers → short-term wins → accelerate → institute change | "Particularly relevant"; the model for SRE teams. Exception: often no need to *create* urgency, since it already exists |
| Prosci ADKAR | Awareness, Desire, Knowledge, Ability, Reinforcement | Limited for operational teams (time constraints), but recommended for SRE **management** coordinating change across globally distributed teams |
| Bridges Transition Model; Kübler-Ross Change Curve | Emotional reactions to change, **not** change-management frameworks | All individual SRE managers should be familiar with both |
| Deming Cycle (PDCA) | Continuous process improvement (e.g., CI/CD adoption) | **Not suited** to organizational change: ignores the human side; iterating on large org changes is counterproductive |

## Concrete numbers (exact, as written)
- Waze acquired by Google in **2013**.
- Message queue crisis: the whole Waze SRE team spent most of a **two-week** period firefighting **24/7**, restarting some queue components **hourly**.
- Guiding coalition: **two SREs and a senior engineer**.
- A dedicated queue was built for one component responsible for **over 30%** of system traffic, which bought a **two-month** window to prototype.
- Old queue handled **tens of thousands of messages per second**. The new system handles **more than 1000 times** the load.
- Last stragglers migrated "a few quarters later."
- Initial microservice tooling prototypes: **one SRE over several days**.
- **More than 95%** of Waze services use Spinnaker for continuous deployment.
- Common tooling: started from **65+** proposed projects, cut to "a handful of vetted topics" after multiple months.
- Virtual team was designed to spend **around 80%** of time on horizontal projects (80/20 project/on-call). This was **abandoned**.
- Efficient co-located group size: **6–10 people**.
- Adoption target: **60% of Google engineers** (their reading of "*almost* everyone at Google").

## Recommended practices and procedures
**Waze (Kotter mapped retroactively):**
1. Use the existing urgency (outages, toil hurting feature velocity) to get leadership to reprioritize.
2. Form a small cross-functional guiding coalition with a strategic vision.
3. Remove barriers with a volunteer army: client teams cut message volume, add compression, split off the heaviest flow.
4. Make client libraries dual-capable (old, new, or both systems) behind a centralized control surface for traffic switching.
5. Migrate low-traffic, high-importance flows first, dual-writing for a backup path. Near misses become short-term wins.
6. Automate migration to accelerate it, then deprecate the old system.
7. Drive adoption by showing value: a centralized release dashboard and standard metrics, volunteer teams first, then a leadership goal, training and consulting.
8. Incentivize standards: bundle new-service creation with the framework so the pipeline setup is trivial.
9. As a thought experiment, look for components that don't scale horizontally or that grow superlinearly against a core business metric (e.g., DAU). Consider redesigning or replacing them.

**Common tooling (ADKAR):**
- Staff with production-experienced engineers **100% dedicated** to software, with schedules and support identical to the rest of Google's software development.
- Build in small, mostly co-located teams. Gather requirements across the org remotely.
- Add product management and UX talent. Treat the work like product management.
- Build an opinionated **zero-config** layer (standard dashboards) on top of the customizable system, and keep custom views for special services.
- Win users bottom-up (no mandate) by proving the new thing is better.
- Communicate transparently and predictably ("We'll have X done by Y date").

## Anti-patterns Google warns against
- Treating change management theory as a waste of time.
- "Best effort" tooling that fits into the cracks: unpredictable delivery and a rotating cast of staff, which erodes trust.
- Virtual, part-time, multi-timezone teams that rotate on-call while building serious software.
- Waiting for a "singing-and-dancing" universal solution, which leads to niche tools proliferating.
- Assuming "if you build something great, people will naturally flock to it."
- Jumping straight to the "perfect" solution.
- Frequent, wrenching org-chart changes, which sap confidence and damage culture.
- Migration costs that aren't small relative to the benefits, so the migration itself becomes the barrier.

## Checklist: what successful common-tooling projects had ("Lessons Learned", Case Study 2)
- [ ] Clearly defined
- [ ] Well advertised in advance
- [ ] Evaluated against many use cases, starting with the **grumpiest adopters**
- [ ] Leaps and bounds better than existing options
- [ ] Adoptable with little to no effort (migration cost near zero, benefits clear)
- [ ] Adoption plan: champions, beta testers, executive sponsors, dedicated engineers
- [ ] Tools offer rapid development, feature stability, common control surfaces, and automatable APIs

Impact questions to ask *after* adoption: How quickly can a new developer build and manage a world-scale service? How easily can an SRE move between domains? How many services can be managed with the same primitives?

## Short verbatim quotes
- "there's no need to *create* a sense of urgency."
- "Incremental change is much easier to manage."
- "migration costs need to be nearly zero (“just recompile and you pick up new $thing”)"
- "Perceptions of how the sausage is made turned out to be more important than we anticipated from the get-go."
- "our first measurement must be adoption."
- ADKAR names the resistance to swapping working systems the "knowledge-to-ability gap."
