# Chapter 30: Embedding an SRE to Recover from Operational Overload

Source: https://sre.google/sre-book/operational-overload/ (SRE Book, Part IV, Management; by Randall Bosetti)

## Core principles

- **Even split** (intro): standard policy splits SRE time evenly between project work and reactive ops work. Ticket volume can upset that balance "for months at a time," which leads to burnout, stalled projects, and worse scalability and reliability.
- **The remedy is one embedded SRE**, placed temporarily. That SRE improves the team's *practices* and does not empty the ticket queue. Sending two "may actually cause problems if the team reacts defensively."
- **Nonlinear scaling** (sidebar "Ops Mode Versus Nonlinear Scaling"): add humans only "as more complexity is added to the system." A team in *ops mode* adds administrators as load grows, for example to manage more VMs. SRE writes software so headcount doesn't grow "as a function of load on the service."
- **Reduce the number of emergencies**, not just the time spent handling them (Identify the Largest Sources of Stress).
- **Team health is a process**, not a heroic effort (Phase 3). Restore the right initial conditions and teach a small set of principles.
- **SLO first** (Start with the Basics): without an SLO, "no other advice in this chapter will be helpful."

## Concrete numbers (exact; the chapter has no formulas or tables)

- Project vs. ops time: split **evenly**. The chapter states no percentage.
- Embed **one** engineer.
- "A 100 request/second service can turn into a 10k request/second service in a year."
- Capacity: the model predicts **2 GB** and a short-term loadtest shows **1.99 GB**. Passing "doesn't necessarily mean that your system capacity is in adequate shape."
- Postmortem anti-example: "Change the streaming timeout back to **60 seconds**."
- Kindling: guide team members through **two or three** issues.

## Recommended practices by phase

### Phase 1: Learn the Service and Get Context
- Explain how habits help or hurt scalability, and point out missed automation and simplification.
- If the team says "my service is tiny," shadow an on-call session to check. A tiny but important service: focus on what blocks reliability work. A new service: prepare the team for explosive growth.
- **Identify the Largest Sources of Stress.** Rank outages by their effect on team stress. Small problems can cause outsized stress.
- **Identify Kindling** (emergencies waiting to happen):
  - New subsystems that aren't self-managing.
  - Knowledge gaps from overspecialization.
  - SRE-built services quietly gaining importance.
  - Reliance on "the next big thing."
  - Frequent alerts nobody diagnoses ("transient"). "Either investigate such alerts fully, or fix the alerting rules."
  - A service with client complaints and no formal SLI/SLO/SLA.
  - Reactive capacity plans ("Add more servers…").
  - Postmortems whose action items only roll back the change.
  - Serving-critical components where SREs say "the devs own it." On-call must at least know what happens when it breaks and how urgent a fix is.

### Phase 2: Sharing Context
- **Write a Good Postmortem for the Team.** Don't comment on old postmortems, because that makes the team defensive. Own the *next* one ("There *will* be an outage while you're embedded"). Write it blamelessly with the on-call engineer, and show that fixes become more permanent.
- Rebut the **Bad Apple Theory**: it is "demonstrably false" ([Dek14], airline safety).
- **Sort Fires According to Type.** Split them into toil (fires that shouldn't exist) and not-toil (part of the job). Build tools for both. Present the list and explain each item: automate it, or accept it as overhead.

### Phase 3: Driving Change
- **Start with the Basics.** Work from the principles in Introduction and Monitoring Distributed Systems, not point by point. If no SLO exists, "get the tech leads and management in a room and start arbitrating."
- **Get Help Clearing Kindling.** Don't fix things yourself, because that reinforces "making changes is for other people." Record extra issues in bugs or docs, since docs are "often the first victim of deadline pressure."
- **Explain Your Reasoning** for every decision, even when nobody asks. "After you leave, the team should be able to predict what your comment on a design or changelist would be."
- **Ask Leading Questions.** These are not loaded questions. They point people back to first principles.

### Conclusion
- Write an **after-action report** with your perspective, examples, explanations, and action items. It can be organized as a **"postvitam"** (in contrast to a postmortem) that explains the critical decisions that led to success.
- Afterward, stay available for design and code reviews. Watch the team "for the next few months" for progress on capacity planning, emergency response, and rollouts.

## Anti-patterns

- Scaling headcount with tickets or load.
- Optimizing how fast you handle emergencies instead of reducing them.
- The embedded SRE fixing issues personally.
- Critiquing archived postmortems.
- Punitive postmortems and Bad Apple thinking.
- Rollback-only action items, for example "Change the streaming timeout back to 60 seconds" instead of "Figure out why it sometimes takes 60 seconds to fetch the first megabyte of our promo videos."
- A loadtest that barely passes treated as proof of capacity.
- Shielding devs from alerts.
- Decisions with no reasoning or poor reasoning, which the team will copy.
- Vague or accusatory questions ("What's up with all of these old, stalled releases?", "Why does the Frobnitzer do so many things?").

## Templates and checklists

**Clearing kindling:**
1. Find useful work one team member can do.
2. Explain how it fixes a postmortem issue *in a permanent way*.
3. Review the code changes and document revisions yourself.
4. Repeat for two or three issues.

**Postmortem prompt:** "Mistakes are inevitable in any system with multiple subtle interactions. You were on-call, and I trust you to make the right decisions with the right information. I'd like you to write down what you were thinking at each point in time, so that we can find out where the system misled you, and where the cognitive demands were too high."

**Good explanations:**
- Push back on a release "because the error budget we set for releases is exhausted," not because the tests are bad.
- Releases must be rollback-safe because a tight SLO requires a small mean time to recovery.

**Good leading question:** "I see that the TaskFailures alert fires frequently, but the on-call engineers usually don't do anything to respond to the alert. How does this impact the SLO?"

**What the embed leaves the team:**
- A technical, possibly quantitative, reason to change.
- A strong example of what change looks like.
- A logical explanation for SRE "folk wisdom."
- Core principles for handling new situations in a scalable way.

## Short verbatim quotes

- "more tickets should not require more SREs"
- "your job is to make the service work, not to shield the development team from alerts."
- "An SLO is probably the single most important lever for moving a team from reactive ops work to a healthy, long-term SRE focus."
- "Leading questions are not loaded questions."
