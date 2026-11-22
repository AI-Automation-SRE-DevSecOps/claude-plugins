# The Site Reliability Workbook, Chapter 6: Eliminating Toil

Source: https://sre.google/workbook/eliminating-toil/ (Challoner, Wijntjes, Huska, Sartwell, Coykendall, Schrier, Looney, Rau, with Beyer, Luebbe, Perry, Suriar). It builds on Chapter 5 of the first SRE book.

## Core principles

- **Working definition (intro):** toil is "the repetitive, predictable, constant stream of tasks related to maintaining a service."
- **The 50% cap (intro):** Google caps the time SRE teams spend on *operational work*, toil and non-toil alike, at **50%**. Note that the cap covers all operational work, not toil alone. The chapter says the exact target may not suit every organization, but having some upper bound still helps.
- **Toil is a spectrum, not a yes/no label ("What Is Toil?", "Toil Taxonomy").** Its traits: **Manual, Repetitive, Automatable, Nontactical/reactive, Lacks enduring value, Grows at least as fast as its source.** A source of toil need not show every trait. Morale is a further consideration.
- Footnote 1: "automatable" is the most subjective trait, and your view of it shifts as you gain automation experience.
- **Fix root causes rather than hiding symptoms** (John Looney sidebar): a cleanup script that hides a kernel logging bug removes the pressure to fix it properly.
- **The best strategy is to engineer toil out at its source** ("Engineer Toil Out of the System").
- **Conclusion:** toil "grows linearly with its complexity and scale" at minimum. Eliminating it is not always worthwhile, so use metrics, ROI, risk assessment and iterative development to decide.

## Concrete numbers, thresholds and formulas

| Item | Value as written | Section |
|---|---|---|
| Cap on operational work | 50% (toil plus non-toil ops) | Intro |
| Example trigger for manual toil | `tmp` directory at 95% utilization | What Is Toil? |
| Dependency math | Each new critical dependency of availability P lowers availability by 1 – P | Troubleshooting for Opaque Architectures |
| Worked example | A four 9s service with nine critical four 9s dependencies becomes three 9s: 0.9999^10 = 99.9% (fn 3) | same |
| Self-service target | "Moving 80–90% of requests to self-service is still a huge reduction" (fn 4) | Provide Self-Service Methods |
| Looney sidebar | More than 20,000 broken machines at any one time; about $1/hour per server; problem cost $1,000/hour; logs over 1 MB truncated | Example: Manual Response |
| Saturn strike policy | 1st failure: reboot the card and reinstall software. 2nd failure: replace the card and return it to the vendor | Case Study 1 |
| Jupiter strike policy | Second failure **within six months** goes straight to a technician for new hardware. Otherwise power-cycle "via two distinct methods" | Case Study 1 |
| Growth | Capacity doubled "almost every 12 months"; Jupiter was more than 6x larger than any previous fabric, with 6x the problems | Case Study 1 |
| Over-analysis | Nearly 3 years (2012–2015) spent on more than 650 memory-error problems. One quarter of acting on the data showed most errors were transient | Don't overthink the problem |
| Clos figure | A small Clos network supports 480 machines below Stage 1 | Fig 6-2 |
| Moonwalk scale | 2.5 billion files, 300 TB, 60,000 POSIX users, 400 volumes, 124 NAS appliances, 60 sites | Case Study 2 |
| Moira outcome | Home directories cut from 65,000 to about 50. Program finished in 2016. Team averaged 3 people. Feature-complete only in phase three (about 2 years in). Long tail of about 800 users. Filers in use 14+ years | Case Study 2 |

## Recommended practices and procedures

**Measuring Toil (3 steps):**
1. **Identify** the toil, ideally with input from stakeholders, including the people who do the work.
2. **Choose a unit of human effort.** Minutes or hours work well; include the cost of context switching. Alternative units: an applied patch, a completed ticket, a manual production change, a predictable email exchange, a hardware operation. The unit must be "objective, consistent, and well understood."
3. **Track continuously** before, during and after the reduction effort, using tools so that measuring does not itself become toil.

**Cost/benefit (Measuring Toil, Fig 6-1, xkcd 1319):** time saved should at least match the time spent building *and maintaining* the automation. A project that looks unprofitable may still be worth doing for indirect benefits: more project work, better morale and less attrition, less context switching, clearer processes, skills growth, less training time, fewer human-error outages, better security, faster responses.

**Toil Taxonomy:** Business Processes (the most common source; ticket toil is "insidious"), Production Interrupts, Release Shepherding, Migrations, Cost Engineering and Capacity Planning, Troubleshooting for Opaque Architectures.

**Toil Management Strategies (as listed):** Identify and Measure Toil (track toil in bugs and rank it by cost to fix versus time saved); Engineer Toil Out of the System; Reject the Toil (consider this first, and batch delayed toil); Use SLOs to Reduce Toil (skip tasks that don't consume the error budget); Start with Human-Backed Interfaces ("engineer behind the curtain"); Provide Self-Service Methods (degrade gracefully to a ticket); Get Support from Management and Colleagues; Promote Toil Reduction as a Feature; Start Small and Then Improve (use metrics such as MTTR); Increase Uniformity (pets vs. cattle); Assess Risk Within Automation; Automate Toil Response (build composable libraries); Use Open Source and Third-Party Tools; Use Feedback to Improve (latency, error rate, rework rate, human time saved).

**Legacy Systems path:** Avoidance, then Encapsulation/augmentation ("refinancing high-interest technical debt into low-interest"), then Replacement/refactoring (incremental, behind a common interface, with canary or blue-green releases and production-sized historical input/output datasets), then Retirement/custodial ownership (stragglers own the remnants).

## Anti-patterns Google warns against

- Scripts that hide a symptom instead of fixing the root cause (Looney sidebar).
- Doing migrations by hand because they are "one-time" (Migrations).
- Copying a human workflow into code step for step, or automation that removes human understanding of what is going wrong (Automate Toil Response).
- UI steps that add overhead: the "Prep component" button gave no progress feedback and fell out of sync with the switch's real state (UIs should not introduce overhead).
- Relying on human expertise. One technician started concurrent drains on every line card awaiting repair, causing congestion and user-visible packet loss (Don't rely on human expertise).
- Monolithic automation (Design reusable components). Over-analysis that blocks action (Don't overthink). Treating automation as "fire and forget" or making it inflexible.
- Automation that creates new toil, such as unnecessary tickets, or that creates problems for other teams (Think holistically).
- Naive read operations at scale, which can spike device load and cause outages (Assess Risk).

## Templates and checklists

**Automation safety checklist (Assess Risk Within Automation):** validate all input in context, including input from upstream systems. Build safeguards equivalent to the indirect alerts a human would get (timeouts, checks on system metrics and current outages), and make monitoring consumable by machines as well as humans. Watch the load created by reads and safety checks. Fall back to human operators when automation hits an unsafe condition.

**Defense in depth (Build in risk assessment):** a secondary check with upper limits on the number of impacted links and impacted devices. Going over either limit automatically opens a tracking bug. Tune the limits to reduce false positives.

**Failure budget:** "establishing an error budget for antitoil automation" (Get a failure budget and manager support).

**Self-service portal requirements (Moira Portal):** a landing page, a continually updated FAQ, the user's share status and usage, and options to request, deactivate, archive, delete, extend or reactivate a share.

**Case Study 2 lessons:** challenge assumptions and retire expensive processes; build self-service interfaces (config-as-code changelists are a cheaper route than a custom portal); start with human-backed interfaces (tickets as a "quick and dirty GUI"); melt snowflakes ("buying the gnome"); use organizational nudges (escalations for new requests, codelabs and cookbooks).

## Short verbatim quotes

- "Experience and intuition are not repeatable, objective, or transferable." (Measuring Toil)
- "rejecting a toil-intensive task should be the first option you consider." (Reject the Toil)
- "Automation should default to human operators if it runs into an unsafe condition." (Assess Risk)
- "Automation craves conformity." (Melt snowflakes)
- "SRE teams must be relentless in eliminating toil, because even if the task seems daunting, the benefits usually exceed the costs." (Conclusion)
