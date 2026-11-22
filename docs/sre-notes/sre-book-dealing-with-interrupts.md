# Digest: SRE Book Chapter 29, "Dealing with Interrupts"

Source: https://sre.google/sre-book/dealing-with-interrupts/ (written by Dave O'Connor, edited by Diane Bates; Part IV, Management)

## Core principles

- **Operational load has three categories** (intro): *pages*, *tickets*, and *ongoing operational activities*.
  - **Pages**: production alerts and their fallout, triggered by production emergencies. They "always have an expected response time (SLO), which is sometimes measured in minutes."
  - **Tickets**: customer requests that require you to take an action. They "may also have an SLO, but response time is more likely measured in hours, days, or weeks."
  - **Ongoing operational responsibilities** (also called "Kicking the can down the road" and "toil"): team-owned code or flag rollouts, ad hoc time-sensitive questions from customers. These may have no defined SLO but still interrupt.
- **Humans are imperfect machines** ("Imperfect Machines"). Interrupt policy has to account for human cost, not only response time. The chapter points out that the usual metrics are "suited to meeting the lowest possible response time, without factoring in more human costs."
- **Cognitive flow state** should be maximized ("Cognitive Flow State"). Flow has these elements: clear goals, immediate feedback, a sense of control, and time distortion. It comes in two kinds:
  - *Creative and engaged*: deep project work.
  - *"Angry Birds"*: flow that comes from doing tasks you already know. On-call can produce flow when it is the only thing you are doing: "interrupts stop being interrupts". The stress comes from mixing on-call with project work, which leaves people in constant "interruptability."
- **Polarize time** ("Polarizing time"): each day a person should know whether they are doing *just* project work or *just* interrupts.
- **Structure belongs to the team, not the individual** ("Do One Thing Well"): the guidance is aimed at managers and influencers. It shapes team structure so that "people aren't set up for failure because of team function or structure." Personal time-management habits are left to each person.

## Concrete numbers and thresholds (exact)

- Page response SLO: "sometimes measured in minutes."
- Ticket SLO: "more likely measured in hours, days, or weeks."
- **Context-switch cost**: "A 20-minute interruption while working on a project entails two context switches; realistically, this interruption results in a loss of a couple hours of truly productive work."
- **Polarized period length**: "Ideally, this time period is a week, but a day or even a half-day may be more practical."
- On-call week: "When an engineer is on-call for a week, that week should be written off as far as project work is concerned."
- Ticket overflow: if tickets exceed what primary plus secondary on-call can close, "structure your ticket rotation to have **two people** handling tickets at any given time."
- The chapter gives **no** formula or numeric cap on interrupt volume. Its threshold is qualitative: "if the volume of interrupts is too high for one person, add another person."

## Metrics for deciding how to handle interrupts ("Factors in Determining How Interrupts Are Handled")

1. Interrupt SLO or expected response time
2. The number of interrupts that are usually backlogged
3. The severity of the interrupts
4. The frequency of the interrupts
5. The number of people available to handle a certain kind of interrupt (e.g., some teams require ticket work before going on-call)

## Recommended practices and procedures

**Managing Operational Load**
- Pages go to a **single** dedicated primary on-call engineer. This minimizes team interruption and avoids the **bystander effect**. The primary may escalate to a teammate when a problem isn't well understood.
- A secondary acts as backup. Sometimes the secondary's only job is to catch fall-through pages, and in that case they may sit on another team.

**General suggestions**
- If an interrupt class exceeds one person, **add another person**. For pages this means the on-call bumps work to the secondary or downgrades pages to tickets.

**On-call**
- The primary focuses *solely* on on-call work. When the pager is quiet, they pick up tickets or other interrupt work that "can be abandoned fairly quickly."
- If a project can't slip by a week, that person shouldn't be on-call. Escalate to get someone else assigned.
- The secondary's role depends on the load. A fall-through-only secondary can probably do project work. If a separate person handles tickets, consider merging that role with the secondary. If the secondary actively helps when pager volume is high, they should be on interrupts too.

**Tickets**
- Make ticket handling a full-time role, for a period one person can manage.
- Don't spread the load across the whole team.

**Ongoing responsibilities**
- Define roles that anyone can pick up, for example a **push manager** who juggles pushes during their on-call or interrupt period.
- **Formalize the handover**, so that nobody shepherds a change for its whole lifetime after their shift ends.

**Reducing Interrupts**
- *Actually analyze tickets*: hand off tickets the way on-call is handed off, to keep shared state. "Lots" of teams do on-call handoffs and page reviews; "very few" do the same for tickets.
- Run a **regular scrub of tickets and pages** by class to find root causes. If a root cause is fixable in a reasonable time, **silence the interrupts until the fix is expected**. This relieves the handler and sets a deadline for whoever is fixing it.
- *Respect yourself, as well as your customers*: use policy as a tool. Push effort back to requesters when a step doesn't need your privileges. For example, the customer prepares the code or config change and sends it to you for review. For flaky, under-resourced, low-value components, consider giving the pager back, deprecating, or replacing.

## Anti-patterns Google warns against

- **Randomly auto-assigning tickets** to team members: "stop," because it is "extremely disrespectful of your team's time."
- Expecting someone to be **on-call and also make progress on projects**, or on anything else with a high context-switch cost.
- Treating an engineer as "an interruptible unit of work, whose context switches are free."
- **Spreading ticket load across the entire team.**
- **Working tickets when not assigned** to look busy. This makes the person less effective and "skew[s] the numbers," which hides an unmanageable queue.
- **Gauntlet-style rotations**: people survive their shift and move on, so root causes never get investigated. This is especially common on large teams where rotations are infrequent.

## Templates and checklists

The chapter has no formal template. The usable checklists are:
- The **five interrupt-handling metrics** listed above.
- The **Fred distraction list** ("Distractibility"): an auto-assigned ticket, an on-call colleague's question, an escalated old ticket, a failing multiweek rollout, direct user questions.
- **Customer request contract**: the request should "be meaningful, be rational, and provide all the information and legwork you need". In return, the team's response "should be helpful and timely."

## Short verbatim quotes

- "A person should never be expected to be on-call and also make progress on projects (or anything else with a high context switching cost)."
- "When you're doing interrupts, your projects are a distraction."
- "silence the interrupts until the root cause is expected to be fixed."
- "Your team sets the level of service provided by your service."
- "It's OK to push back some of the effort onto your customers."
- "Policy can be as powerful a tool as code."
