# Chapter 34 — Conclusion (Site Reliability Engineering, Google, 2016)

Source: https://sre.google/sre-book/conclusion/
Author: Benjamin Lutch (VP, Site Reliability Engineering, Google), edited by Betsy Beyer. Part V — Conclusions.

Note on structure: the chapter is a short essay with **no subsection headings**. Section references below describe the order of its passages: (1) personal opening & SRE growth, (2) the dual nature of SRE work, (3) future-proof axioms and the two dynamics of growth, (4) the aircraft/cockpit analogy, (5) the goal statement for SRE teams. It contains **no formulas, thresholds, tables, or checklists**. The concrete figures it gives are descriptive, not prescriptive.

## Core principles

- **SRE's success comes from its principles.** Lutch credits "the nature of the principles by which it operates" for SRE's ability to scale the infrastructure "in an intelligent, efficient, and scalable way" (passage 1).
- **Two equally important types of work** (passage 2): SREs staff on-call shifts, getting hands-on with systems and seeing "where and how these systems break" and how to scale them, *and* they have time to reflect and decide what to build so the systems are easier to manage. SREs play "both the roles of the pilot *and* the engineer/designer."
- **Codify operational experience in code.** What SREs learn from running infrastructure is "codified in actual code and then packaged as a discrete product." Other SRE teams, the rest of Google, and outside users (Google Cloud is the example) can then reuse it (passage 2).
- **Build on general, durable axioms.** A team or system should rest on "rules or axioms that are general enough to be immediately useful, but that will remain relevant in the future." The responsibilities Ben Treynor Sloss set out in the Introduction are called "flexible, mostly future-proof" and "spot-on 10 years after they were conceived" (passage 3).
- **Two dynamics of growth** (passage 3):
  1. *Constant core concerns:* even when systems are "1,000 times larger or faster," they must still be reliable, flexible, easy to manage in an emergency, well monitored, and capacity planned.
  2. *Evolving activities:* the work itself changes as services and SRE skills mature.
- **Operator headcount should not grow with system complexity** (passage 4, the aircraft analogy). Planes went from single-engine, fragile, non-redundant aircraft to the 747, yet the cockpit still holds "just two pilots." The reasons given:
  - the interfaces are well designed and approachable enough that normal operation can be learned;
  - they are flexible enough, and operators trained well enough, that emergency response is "robust and quick";
  - the cockpit was designed by people who understand complex systems and how to present them to humans "in a way that's both consumable and scalable."
  - The systems behind the cockpit share the properties the book covers: availability, performance optimization, change management, monitoring and alerting, capacity planning, and emergency response.

## Concrete numbers (descriptive, exactly as written)

| Figure | Context |
|---|---|
| "a few hundred engineers" (2006) → "over 1,000 people today" | Size of Google SRE from when Lutch joined until publication |
| "a dozen sites" | Number of sites SRE is spread over |
| "10 years" | How long Treynor Sloss's responsibilities have stayed valid |
| "1,000 times larger or faster" | Hypothetical system growth that leaves core concerns unchanged |
| "20 machines" → "tens of thousands of machines" | Example of how SRE work has evolved |
| "a hundred years" ago; "6,000 miles"; "two pilots" | Aircraft analogy |

None of these is an SLO, a threshold, or a staffing ratio. The book's quantitative guidance, such as the 50% cap on ops work, lives in other chapters (e.g., Ch. 1, Ch. 5).

## Recommended practices

- Split SRE time between hands-on operations (on-call) and engineering work that makes systems easier to manage. The two are "equally important."
- Turn operational lessons into reusable software products rather than leaving them as tribal knowledge.
- Raise the level of abstraction as scale grows. The example: "build a dashboard for 20 machines" becomes "automate discovery, dashboard building, and alerting over a fleet of tens of thousands of machines."
- Design operator interfaces (APIs, tooling) that are easy to learn for normal operation and flexible enough for emergencies. Pair them with well-trained operators.
- **Goal statement (passage 5):** an SRE team should be "as compact as possible and operate at a high level of abstraction," relying on "lots of backup systems as failsafes and thoughtful APIs." It must still keep comprehensive, day-to-day knowledge of how the systems operate, fail, and should be responded to.

## Anti-patterns (implied by contrast; not listed explicitly)

- **The early-aviation model:** one person acts as pilot, mechanic, and cargo loader, and the systems are "essential, simple, and fragile, and most likely were not redundant." Failure of any part is "catastrophic," and repairs happen in flight.
- Growing the operations team in step with system size, instead of scaling through abstraction and automation.
- Operating only at a high level of abstraction while losing hands-on knowledge of how systems fail. The chapter asks for both compactness *and* deep knowledge.
- Doing only on-call or only engineering. The model depends on doing both.

## Templates / checklists

None in this chapter. (The next items in the book, Appendices A–F, hold the availability table, best practices, incident-state document, postmortem, launch checklist, and production meeting minutes.)

## Short verbatim quotes

- "SRE teams are constructed so that our engineers divide their time between two equally important types of work."
- "we have the pleasure of playing both the roles of the pilot and the engineer/designer."
- "Our experiences running massive computing infrastructure are codified in actual code and then packaged as a discrete product."
- "they still need to remain reliable, flexible, easy to manage in an emergency, well monitored, and capacity planned."
- "But take a look into the cockpit and what do you find? Just two pilots again!"
- "An SRE team should be as compact as possible and operate at a high level of abstraction"
