# Chapter 33: Lessons Learned from Other Industries

Source: https://sre.google/sre-book/lessons-learned/. Written by Jennifer Petoff, edited by Betsy Beyer. This is the first chapter of Part V, "Conclusions."

The chapter is comparative, not procedural. It is built on interviews with Googlers who used to work in defense, lifeguarding, LASIK and medical devices, telecom/E911, avionics and railway signaling, synthetic-diamond manufacturing, proprietary trading, the UK civil nuclear industry, the US nuclear Navy, and air traffic control. It sorts SRE principles into **four key themes** and shows how each one appears in those industries.

## Core principles (the four themes)

1. **Preparedness and Disaster Testing.** "Hope is not a strategy." Google runs annual **DiRT** (Disaster and Recovery Testing) drills that "inflict actual outages" to (a) ensure systems react the way we think they will, (b) find unexpected weaknesses, and (c) find ways to make systems more robust and prevent uncontrolled outages.
2. **Postmortem Culture.** Blameless postmortems are the SRE form of **CAPA** (Corrective and Preventive Action): systematic investigation of root causes to prevent recurrence. The main difference between industries is the *motivation* behind postmortems: regulation, or safety.
3. **Automating Away Repetitive Work and Operational Overhead.** SREs are "software engineers with a low tolerance for repetitive reactive work." Automation lowers overhead and frees time for proactive improvement.
4. **Structured and Rational Decision Making.** Decisions are data-driven, their criteria are agreed in advance, and they are made without deference to the HiPPO.

The conclusion: Google has "a higher appetite for velocity" than most industries. That is acceptable because lives are generally not directly at risk, so tools such as **error budgets** can "fund" innovation and calculated risk-taking. When the stakes are high, "a conservative approach to achieving high reliability is warranted."

## Concrete numbers, thresholds, standards (as written)

- **UK nuclear rule of thumb:** if a plant must respond to a situation in **less than 30 minutes**, that response **must be automated**.
- **US nuclear Navy live drills:** "carried out religiously, every week, two to three days per week."
- **Defense design culture (Peter Dahl):** "an entire year of design, followed by just three weeks of writing the code."
- **Alcoa (Paul O'Neill):** staff had to notify the CEO **within 24 hours** of any injury that lost a worker day.
- **Knight Capital, 2012:** a "software glitch" lost **$440M** in "just a few hours."
- **2010 Flash Crash:** losses "on the magnitude of trillions of dollars in just 30 minutes." The chapter text blames "a rogue trader attempting to manipulate the market with automated means". Footnote 161 cites a headline blaming a "computer algorithm."
- **Telecom:** long-distance switches from the **1980s** are still in use because they are "pretty much bulletproof and massively redundant."
- **Celebrity phone-number leak (2005):** thousands of simultaneous callers produced symptoms "similar to a DDoS or massive routing error."
- **Software safety standards named:** UK Defence Standard 00-56, IEC 61508, IEC513 (spelled this way in the text), US DO-178B/C, DO-254. Reliability levels: **Safety Integrity Level (SIL) 1–4**.
- **Regulators named:** FCC (telecom), FAA (aviation), OSHA (manufacturing/chemical), FDA (medical devices), EU National Competent Authorities.
- **Near-miss formula (VM Brasseur, YAPC NA 2015):** "Latent error, plus an enabling condition, equals things not working quite the way you planned."

## Recommended practices and procedures

**Preparedness and Disaster Testing.** The chapter lists seven strategies from other industries:
- *Relentless organizational focus on safety.* In manufacturing, "every management meeting started with a discussion of safety". Workers must feel empowered to speak up.
- *Attention to detail.* In the nuclear Navy, routine maintenance (e.g., lube oil) keeps small issues from snowballing in interconnected systems.
- *Swing capacity.* Telecom keeps a SOW ("switch on wheels"), a mobile telco office, for emergencies or known events (e.g., the Olympics).
- *Simulations and live drills.* The potential consequences of an outage decide which one fits. Aviation uses extremely realistic simulators with live data feeds because it cannot test in production. Telecom runs live hurricane drills, which led to weatherproof facilities with generators inside the building. The nuclear Navy combines "what if" thought exercises with controlled live drills, because thought exercises alone are "not sufficient." Lifeguards stage "mystery shopper"-style mock drownings that look like real emergencies.
- *Training and certification.* Lifeguards certify and then recertify periodically. Training covers fitness, technical skills (first aid/CPR), operational and team-response skills, plus **site-specific** training.
- *Detailed requirements gathering and design.* For medical devices, gather requirements from the actual users and maintainers (surgeons, technicians).
- *Defense in depth (and breadth).* Nuclear reactors have redundancy on all systems, mandated fallbacks behind primary systems, and multiple layers of protection ending in a final physical barrier.

**Postmortem Culture.**
- Evaluate four things: **what happened; the effectiveness of the response; what we would do differently next time; what actions will be taken to make sure it doesn't happen again.**
- Focus on *what* went wrong, not *who*. Publish postmortems across SRE teams.
- Scrutinize **near misses** as "a type of preemptive postmortem". The UK's CHIRP takes confidential near-miss reports and publishes analyses in newsletters.
- Lifeguards: write-ups after any incident, end-to-end team review of serious ones, operational changes and training afterward, and a counselor after traumatic incidents.

**Automation.** Industries differ in where they draw the line:
- Manufacturing automates for efficiency, repeatability and tighter tolerances.
- Aviation automates failover but has humans verify other tasks and manually inspect ATC implementations.
- LASIK: automated sanity checks flag out-of-range manual input, and iris matching against a photo from the pre-op test eliminated patient mix-ups. "An entire class of medical errors disappeared."

**Structured decision making.** Four criteria:
- The basis for the decision is agreed in advance, not justified ex post facto.
- The inputs are clear.
- Assumptions are stated explicitly.
- Data beats feelings, hunches, or the opinion of the most senior person in the room.

Baseline assumption: everyone has users' best interests at heart and can figure out how to proceed from the data. Other examples from the chapter:
- Manufacturing runs controlled experiments with statistical significance.
- Trading uses a separate **enforcement team** that halts trading on any abnormality, and only that team can bring the system back up.

## Anti-patterns and cautions

- Assigning blame. "Dwelling on *who* might have caused the outage is counterproductive."
- Ignoring near misses. They "are effectively disasters waiting to happen."
- Fast automation that is misconfigured. "Speed can be a negative if these tasks are configured incorrectly" (Knight Capital, Flash Crash). The nuclear Navy avoided automation in favor of interlocks and a "trusted human decision chain."
- HiPPO-driven decisions.
- Relying only on thought exercises without practice. Responses must be practiced "so they are not forgotten."
- Not exactly an anti-pattern, but it fits only some contexts: "if it ain't broke, don't fix it…ever" and the playbook/"the binder" approach. These suit slowly evolving industries or ones with limited worker skill levels, not Google's high rate of change.

## Templates and checklists

- **DiRT goals** (3 items) and **postmortem evaluation** (4 items), listed above.
- **Decision-making criteria** (4 items), listed above.
- **Preparedness strategies** (7 items): safety focus, attention to detail, swing capacity, simulations/live drills, training/certification, requirements/design, defense in depth.

## Short verbatim quotes

- "Hope is not a strategy."
- "If a lifeguard's feet go in the water, there will be paperwork!"
- "If we aren't trading, we aren't losing money." (John Li)
- "Near misses are effectively disasters waiting to happen."
- "Computers can execute tasks very quickly, and speed can be a negative if these tasks are configured incorrectly."
