# Chapter 5 — Eliminating Toil (Google SRE Book)

Source: https://sre.google/sre-book/eliminating-toil/ · Written by Vivek Rau, edited by Betsy Beyer.
Sections: Toil Defined · Why Less Toil Is Better · Calculating Toil · What Qualifies as Engineering? · Is Toil Always Bad? · Conclusion.

Opening epigraph (Carla Geisser): "If a human operator needs to touch your system during normal operations, you have a bug. The definition of normal changes as your systems grow."

## Core principles

- **Toil is a specific term, not "operational work" in general.** It is *not* "work I don't like to do," and it is *not* administrative chores or grungy work. (Toil Defined)
- **Overhead is separate from toil.** Overhead is administrative work not directly tied to running a production service: team meetings, setting and grading goals (OKRs), snippets, HR paperwork. (Toil Defined)
- **Grungy work with long-term value is not toil.** Example: cleaning up the entire alerting configuration for a service and removing clutter. (Toil Defined)
- **Definition:** toil is work "tied to running a production service" that tends to be manual, repetitive, automatable, tactical, devoid of enduring value, and scales linearly with service growth. A task need not have all attributes; the more it matches, the more likely it is toil. (Toil Defined)
- **Engineering** is novel, intrinsically requires human judgment, produces a permanent improvement, is guided by a strategy, and lets a team handle a larger service or more services "with the same level of staffing." (What Qualifies as Engineering?)
- Reducing toil and scaling services is the "Engineering" in SRE; it lets SRE scale **sublinearly** with service size. (Why Less Toil Is Better)
- Toil is not always bad in small doses; it "becomes toxic when experienced in large quantities." (Is Toil Always Bad?)

## The six toil attributes (Toil Defined)

| Attribute | Chapter's test |
|---|---|
| Manual | Hands-on human time running even an automating script counts as toil — the **hands-on time, not elapsed time**. |
| Repetitive | First or second time doing a task is *not* toil; novel problems are not toil. |
| Automatable | If a machine could do it as well, or the need could be designed away, it is toil. Human judgment being essential suggests it's not toil (with a caveat, see footnote 21). |
| Tactical | Interrupt-driven and reactive, not strategy-driven. "Handling pager alerts is toil." |
| No enduring value | Service is in the same state after the task. |
| O(n) with service growth | Work scales linearly with service size, traffic, or user count. |

## Concrete numbers, thresholds, formulas

- **50% cap:** "advertised goal of keeping operational work (i.e., toil) below 50% of each SRE's time." At least 50% should be engineering project work that reduces future toil or adds service features. (Why Less Toil Is Better)
- The 50% engineering target is measured **"when averaged over a few quarters or a year"**; toil is spiky, so some quarters may dip below. Long-haul averages significantly below 50% mean the team must "step back and figure out what's wrong." (What Qualifies as Engineering?)
- **Growth target:** an ideally managed service "can grow by at least one order of magnitude with zero additional work, other than some one-time efforts to add resources." (Toil Defined)
- **On-call floor formula:** typical SRE has one week primary + one week secondary on-call per cycle, so lower bound on toil = 2 / rotation size:
  - 6-person rotation: 2/6 = **33%**
  - 8-person rotation: 2/8 = **25%** (Calculating Toil)
- **Survey data:** quarterly surveys show average toil about **33%**; outliers range from **0%** (pure dev, no on-call) to **80%**. (Calculating Toil)
- **Top sources of toil, in order:** (1) interrupts (non-urgent service-related messages and emails), (2) on-call (urgent) response, (3) releases and pushes. (Calculating Toil)

## Categories of SRE work (What Qualifies as Engineering?)

1. **Software engineering** — writing/modifying code plus design and docs: automation scripts, tools/frameworks, scalability/reliability features, hardening infrastructure code.
2. **Systems engineering** — configuring production systems or documenting them for lasting improvement from one-time effort: monitoring setup, load balancing config, server config, OS tuning; also consulting on architecture, design, productionization for dev teams.
3. **Toil** — work directly tied to running a service that is repetitive, manual, etc.
4. **Overhead** — admin work not tied to running a service: hiring, HR paperwork, team/company meetings, bug queue hygiene, snippets, peer reviews and self-assessments, training courses.

Note: bug queue hygiene is classed as **overhead**, not toil.

## Recommended practices & procedures

- Keep toil under 50% per SRE; measure (Google uses quarterly surveys). (Calculating Toil)
- When individuals report excessive toil, managers should **spread toil load more evenly** across the team and encourage those SREs to find satisfying engineering projects. (Calculating Toil)
- Prefer engineering that is design-driven and generalized ("the more generalized, the better"). (What Qualifies as Engineering?)
- If burdened by too much toil, "be very concerned and complain loudly." (Is Toil Always Bad?)
- "Eliminate a bit of toil each week with some good engineering." (Conclusion)
- Keep the hiring promise: don't let SRE or any subteam "devolve into an Ops team." (Why Less Toil Is Better)

## Anti-patterns Google warns against

- **Mislabeling judgment-heavy alerts as "not toil"** (footnote 21): a service that pages SREs several times a day, each needing complex judgment, is poorly designed. It must be simplified/rebuilt to eliminate failure conditions or handle them automatically; until then, that judgment work "is definitely toil."
- **Letting toil expand unchecked** — it "can quickly fill 100% of everyone's time." (Why Less Toil Is Better)
- **Counting script-running as not-toil** because a script exists (Manual attribute).
- **Too much toil — individual harms:** career stagnation ("you can't make a career out of grunge"), low morale (burnout, boredom, discontent).
- **Too much toil — organizational harms:** creates confusion about SRE's engineering role; slows progress (feature velocity drops); sets precedent (Devs shift ops work to SRE); promotes attrition (best engineers leave); causes breach of faith (new hires promised project work feel cheated). (Is Toil Always Bad?)

## Checklist: is this task toil?

- [ ] Tied to running a production service? (If no → overhead, not toil)
- [ ] Manual hands-on human time?
- [ ] Done repeatedly (beyond the 1st/2nd time)?
- [ ] Could a machine do it, or could design remove the need?
- [ ] Interrupt-driven / reactive?
- [ ] Service unchanged afterward?
- [ ] Scales linearly with size/traffic/users?
More "yes" answers → more likely toil. If it produces a permanent improvement, it probably isn't toil even if grungy.

## Short verbatim quotes

- "Toil is not just 'work I don't like to do.'"
- "An ideally managed and designed service can grow by at least one order of magnitude with zero additional work"
- "toil tends to expand if left unchecked and can quickly fill 100% of everyone's time."
- "Toil becomes toxic when experienced in large quantities."
- "Let's invent more, and toil less."
