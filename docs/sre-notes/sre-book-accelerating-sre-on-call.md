# SRE Book Ch. 28 — Accelerating SREs to On-Call and Beyond

Source: https://sre.google/sre-book/accelerating-sre-on-call/ (Part IV – Management). Author: Andrew Widdowson; editor: Shylaja Nukala. Subtitle: "How Can I Strap a Jetpack to My Newbies While Keeping Senior SREs Up to Speed?"

## Core principles
- **Trust is the foundation** (*You've Hired Your Next SRE(s), Now What?*). On-callers must trust that teammates know how the system works, can diagnose atypical behavior, ask for help, and act under pressure. Asking "what does a newbie need to learn to go on-call?" is "essential, but not sufficient". You also have to ask how existing on-callers will judge readiness and how the team benefits from what newbies bring.
- **No single training style or "magic formula."** Use a mix of abstract/passive and applied/active learning to suit different learning styles.
- **Structure over chaos** (*Initial Learning Experiences*). Contain reactive work through proactive work, and that applies to onboarding too. Front-load abstract concepts that come up again and again, and give hands-on experience as early as practical.
- **What kind of engineer to create** (*Creating Stellar Reverse Engineers and Improvisational Thinkers*): strong **reverse engineers**, people who **think statistically** rather than procedurally, and people who can **improvise fully** when SOPs break down.
- **"Able to take on-call" is a proxy** for "knows enough and can figure out the rest." The chapter says being on-call is *not* the single most important purpose of an SRE.
- **Learning continues after on-call.** Systems change, so the team's operational knowledge goes stale ("historic at best").
- Closing charge: SREs "have to scale your humans faster than you scale your machines."

## Concrete numbers, structures, tables (as written)
- **Table 28-1, SRE education practices** (recommended pattern vs. anti-pattern):
  | Recommended | Anti-pattern |
  |---|---|
  | Concrete, sequential learning experiences | Deluging students with menial work (alert/ticket triage); "trial by fire" |
  | Reverse engineering, statistical thinking, fundamental principles | Training strictly through operator procedures, checklists, playbooks |
  | Celebrating failure analysis; suggesting postmortems to read | Treating outages as secrets buried to avoid blame |
  | Contained but realistic breakages fixed with real monitoring/tooling | First chance to fix something comes only after already on-call |
  | Group role-play of theoretical disasters | Experts whose techniques/knowledge are compartmentalized |
  | Shadowing the on-call rotation early, comparing notes | Pushing students into primary on-call before holistic understanding |
  | Pairing students with experts to revise targeted sections of the training plan | Treating on-call training plans as static, editable only by SMEs |
  | Nontrivial project work giving partial ownership of the stack | All new project work to the most senior SREs; juniors get "scraps" |
- **Figure 28-1 blueprint.** The x-axis runs from abstract to applied work. The y-axis is time, read top-down. Activities at or after going on-call have dashed lines because learning there becomes self-directed. "Project work & ownership" is drawn as a triangle: it starts small and grows.
- **Disaster role play:** weekly. Each session runs **30–60 minutes**. The GM picks **two** people to be primary and secondary on-call.
- **"Let's burn a search cluster to the ground!"** is run **quarterly**.
- **Reverse-engineering class scenario:** the Google News team has been unreachable for **30 days** (they went on a Bermuda Triangle cruise).
- **Shadow on-call:** pages are copied to the newbie, **at first only during business hours**.

## Recommended practices & procedures
- **Cumulative, orderly learning paths.** Group topics by purpose or by normal-case order of execution. Example curriculum for a serving stack: 1) how a query enters the system (networking, datacenter, frontend LB, proxies), 2) frontend serving (app frontends, query logging, user-experience SLOs), 3) mid-tier services (caches, backend LB), 4) infrastructure (backends, compute), 5) tying it together (debugging, escalation, emergency scenarios).
- **On-call learning checklist** (Google Search SRE). It lists expert contacts, key docs, the knowledge you must have before moving on, and comprehension questions. It deliberately does *not* encode procedures or playbooks, which keeps it "relatively future-proof." Homework answers are checked by a mentor and gate progress to the next phase.
- **Tiered access ("powerups").** Start with read-only access. Mutating production comes later. Each completed checklist section earns deeper access.
- **Targeted starter projects** (split time between learning *and* project work):
  - Ship a trivial user-visible feature all the way through to production.
  - Add monitoring where there are blind spots.
  - Automate a pain point that isn't yet painful enough to have been automated.
- **Reverse Engineering a Production Service (without help from its owners).** Trace a browser query through the infrastructure ("follow the RPC"; this also works for batch systems, where you start from the triggering event). Use several discovery methods, since the first assumption about the entry path is too narrow. Use self-reported RPC connectivity plus white-box and black-box monitoring. Afterwards each student diagrams a stack from their home team and presents it to a senior SRE.
- **Five Practices for Aspiring On-Callers:**
  1. **Read and share postmortems.** Curate "teachable postmortems." Run "postmortem reading clubs" and "tales of fail" sessions. Ask related teams to publish their best ones.
  2. **Disaster role playing** ("Wheel of Misfortune"/"Walk the Plank"). The GM plays other teams during escalation, steers participants away from red herrings, and adds urgency with new information or pointed questions ("How could you stop the bleeding in the short term?").
  3. **Break real things, fix real things.** Use a multihomed instance taken off live traffic, or a fully featured staging/QA instance, under synthetic load. Proctors introduce faults such as misconfigs, memory leaks, regressions, crashing queries, and storage bottlenecks. Alternative: start from a known-good config and gradually impair bottlenecks while watching upstream and downstream effects.
  4. **Documentation as apprenticeship.** Before new hires arrive, sort checklist sections by how up to date they are. Assign each newbie to overhaul **one or two** of the most outdated sections. Their revision must be peer-reviewed by one or more of the senior SREs listed as experts.
  5. **Shadow on-call early and often.** Shadow only after finishing the fundamentals (e.g., the checklist). Shadow several team members. Share a terminal or sit together, and review the reasoning afterwards. An optional last step is **"reverse shadow"**: the newbie is primary and the expert diagnoses independently *without modifying state*.
- **Gate going on-call** with a final exam or with a completed checklist. Celebrate it as a team rite of passage.
- **Continuing education.** Run a regular learning series where the SREs shepherding upcoming changes present them, with developers co-presenting as needed. Record the talks to build a library. SREs should also give talks to developers.

## Anti-patterns Google warns against
- **"Trial by fire" onboarding.** In the John/FooServer example the new hire is given every incoming ticket on day one and told it will eventually "click." This alienates capable engineers. "If the set of work one encounters in a tickets queue will adequately provide training for said job, then this is not an SRE position."
- Unclear progress ("What am I working on? How much progress have I made? When will I be ready?") hurts development and retention, especially for introspective people.
- Being too procedural during outages, and carrying too many untested assumptions.
- Handing postmortem writing *solely* to the newbie. They should be a **coauthor**, so postmortems don't come to look like grunt work.
- Every other anti-pattern in Table 28-1.

## Templates / checklists
**On-call learning checklist section (example):** Results Mixing Server ("Mixer")
- Frontended by: Frontend server. Backends called: Results Retrieval Server, Geolocation Server, Personalization Database.
- SRE experts; Developer contacts.
- Know before moving on: which clusters run Mixer; how to roll back a release; which backends are "critical path" and why.
- Read and understand: overview sections, rollout playbook, performance analysis.
- Comprehension questions: e.g., "How can you fix a bad push of the geolocation dataset?"

**Homework questions:** Which backends are in the critical path, and why? What could be simplified or automated? Where is the first bottleneck, and how would you relieve it if it saturated?

**Burn-a-cluster procedure:** (1) discuss which observable characteristics will change; (2) poll participants for predictions and their reasoning before inflicting the damage; (3) validate assumptions and justify the behavior observed.

**Who the checklist serves:** the student (system boundaries, what matters most); mentors and managers (visible progress: "What sections are the most confusing?"); all team members (a "social contract" for joining on-call).

## Short verbatim quotes
- "as SRE, you have to scale your humans faster than you scale your machines."
- "If the set of work one encounters in a tickets queue will adequately provide training for said job, then this is not an SRE position."
- "keep in mind that its most appreciative audience might be an engineer who hasn't yet been hired."
- "it's unfair to make newbies wait until their first real page to have a chance to learn and retain knowledge."
- "going on-call is a rite of passage and it should be celebrated as a team."

Note: the text refers back to the Mixer checklist example as "Table 28-2", but on the web page it appears inline without that label.
