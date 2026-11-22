# SRE Book, Chapter 7: The Evolution of Automation at Google

Source: https://sre.google/sre-book/automation-at-google/. Written by Niall Murphy with John Looney and Michael Kacirek; edited by Betsy Beyer.

## Core principles

- **Automation is a force multiplier, not a panacea.** Software automation beats manual operation "in most circumstances," but better than either is "a higher-level system design requiring neither of them—an *autonomous* system." (intro)
- **The Value of Automation.** The chapter names five values: *Consistency*, *A Platform*, *Faster Repairs*, *Faster Action*, *Time Saving*.
  - Consistency: for "the execution of well-scoped, known procedures," consistency is "in many ways the primary value of automation."
  - Platform: it "centralizes mistakes" (fix a bug once and it stays fixed), extends to new tasks, and exports metrics. Having no automation is "a tax levied on the operation of a system."
  - Faster Repairs: lower MTTR for common faults. Faster Action: procedures should be "scoped over well-defined domains."
  - Time Saving: cited most often, but the hardest to measure. "Decoupling operator from operation is very powerful."
- **The Value for Google SRE.** At planet scale, "consistency, quickness, and reliability dominate." Google built its own APIs where vendors offered none. It is not appropriate to automate every component; the goal is to build platforms, or to *position* yourself to build them.
- **Automation is "meta-software"**, meaning software that acts on software (The Use Cases for Automation). Google SRE mostly automates the *lifecycle of systems, not their data*.
- **Abstraction trade-off.** Higher-level abstractions (Chef, Puppet) are easier to reason about than POSIX-level tools (Perl), but a "leaky abstraction" makes you fail "systemically, repeatedly, and potentially inconsistently." Example: a binary push that is assumed to be atomic.
- **Reliability Is the Fundamental Feature.** A large enough system *must* repair itself, which takes "some capacity for self-introspection," exposed to humans as well.

## A Hierarchy of Automation Classes (the 5 levels, exact example)

1. No automation: the database master is failed over manually between locations.
2. Externally maintained system-specific automation: an SRE has a failover script in his or her home directory.
3. Externally maintained generic automation: the SRE adds database support to a "generic failover" script that everyone uses.
4. Internally maintained system-specific automation: the database ships with its own failover script.
5. Systems that don't need any automation: the database notices problems and fails over automatically without human intervention.

The ideal is a system that "needs *no glue logic at all*," handling "first order" manipulations (adding accounts, turnup) inside the application. The turnup case study restates the path, with level 2 as "operator-written" automation and level 5 as "autonomous systems that need no human intervention."

## Concrete numbers (as written)

**Automate Yourself Out of a Job (Ads MySQL on Borg)**
- Ads DB was "mature and managed" from 2005 to 2008. A proof of concept on Borg ran in late 2008. The "Decider" daemon was completed in 2009.
- Borg tasks move "as frequently as once or twice per week." That rate was tolerable for replicas and unacceptable for masters.
- Manual master failover took **30–90 minutes per instance**. With manual failovers, best-case availability was **99% uptime**, which fell short of business requirements.
- To meet error budgets, each failover had to cause **less than 30 seconds of downtime**.
- Decider completed planned and unplanned failovers in **<30 seconds 95% of the time**.
- Time spent on mundane operational tasks fell **by 95%**. Once schema changes were automated too, the total operational maintenance cost fell by **nearly 95%**. About **60% of hardware** was freed through bin-packing.

**Soothing the Pain: Applying Automation to Cluster Turnups**
- Bigtable incident: the first (logging) disk on **12-disk** systems was deliberately left unused. A year later, automation read "first disk unused" as "no storage configured" and wiped the cluster.
- Turnup took "six or more weeks" from network-ready to serving. Management's mission: five clusters reaching network-ready on the same day, in three months, each to be turned up in one week.
- Idempotent fixes could run **every 15 minutes**. With them, a small team went from network-ready to "Serving 1% of websearch and ads traffic" in a week or two.
- More than **a thousand separate changes a day** hit the affected systems. Rate of change, teams and services "seem to double each year."

**Automation: Enabling Failure at Scale (Diskerase)**: recovery took "the better part of two days" of reinstalling machines, followed by weeks of auditing.

## Recommended practices & procedures

- **Prodtest (Detecting Inconsistencies with Prodtest):** extend the Python unit test framework to test *real-world services*. Tests are chained by dependencies, and one failure aborts everything downstream. When another team's misconfiguration delays you, file a bug to extend their Prodtest.
- **Resolving Inconsistencies Idempotently:** pair each test with a fix (e.g., `TestDnsMonitoringConfigExists` → `FixDnsMonitoringCreateConfig`, then `TestDnsMonitoringConfigPushed` → `FixDnsMonitoringPushConfig`). Fixes must be idempotent and may assume their dependencies are met. "If a fix fails multiple times, the automation assumes that the fix failed and stops, notifying the user."
- **Least privilege and audit:** replace SSH/root with ACL-driven, RPC-based Admin Servers, gated on code review, that log the requestor, parameters and results of every RPC.
- **Service-Oriented Cluster-Turnup:** service owners own an Admin Server that handles turnup and turndown RPCs and publish an API contract, while staying free to change the implementation underneath. The result is "low-latency, competent, and accurate."
- **Three dimensions to evaluate automation (The Inclination to Specialize):** *Competence* (accuracy), *Latency* (how quickly all steps execute), *Relevance* (the proportion of the real-world process that is covered).
- **Recommendations:** the highest leverage is "in the design phase." Use decoupled subsystems, introduce APIs, and minimize side effects. You do not need to be at Google scale: automation is worth more than a time-spent versus time-saved calculation suggests.
- After Diskerase: add sanity checks, **rate limiting**, and idempotent decommission workflows.

## Anti-patterns Google warns against

- Turnup automation maintained apart from the core system, which leads to "bit rot." Critical automation that runs rarely (such as cluster failover every few months) is fragile because its feedback cycle is long.
- Relying on implicit "safety" signals (the Bigtable wipe).
- Free-form SSH scripts, which became "a cholesterol of technical debt."
- Test, then fix, then retest with latency between the steps, which produced flaky tests. Fixes that are not idempotent can leave the system in an inconsistent state.
- Handing automation to a separate turnup team, which creates bad incentives: nobody pays down the service owners' debt, and PMs choose features over automation.
- Treating an empty set as a special value meaning "everything" (Diskerase sent nearly all colo machines to be erased).
- Automation that replaces manual actions while assuming those manual actions stay available. Operators lose practice and their mental models drift from reality (Air France 447, Bainbridge's "ironies"). Drills help.

## Checklists

Questions to answer before declaring a service ready for customer traffic (Prodtest section):
- Were all of the service's dependencies available and correctly configured?
- Were all configurations and packages consistent with other deployments?
- Could the team confirm that every configuration exception was desired?
## Short quotes

- "For SRE, automation is a force multiplier, not a panacea."
- "Automation needs to be careful about relying on implicit 'safety' signals."
- "Automation code, like unit test code, dies when the maintaining team isn't obsessive about keeping the code in sync…"
- "The most functional tools are usually written by those who use them."
- "Reliability is the fundamental feature, and autonomous, resilient behavior is one useful way to get that."
