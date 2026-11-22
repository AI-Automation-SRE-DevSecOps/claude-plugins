# SRE Book, Chapter 13: Emergency Response

Source: https://sre.google/sre-book/emergency-response/ (written by Corey Adam Baye, edited by Diane Bates). The chapter is built around three Google case studies (test-induced, change-induced, process-induced), each laid out as Details / Response / Findings ("What went well" / "What we learned"). It then closes with general lessons. Postmortem mechanics are in Ch. 15 and incident process in Ch. 14 (Managing Incidents); this chapter does not define them.

## Core principles

- **Don't panic; escalate early** ("What to Do When Systems Break"). If you feel overwhelmed, bring in more people. "Sometimes it may even be necessary to page the entire company." If your organization has an incident response process, know it and follow it.
- **A good response comes from preparation.** It "takes preparation and periodic, pertinent, hands-on training." It needs support from the board and management, so that teams can spend "money, time, energy, and possibly even uptime" on readiness.
- **Mitigate quickly first** ("All Problems Have Solutions"). "The highest priority is to resolve the issue at hand quickly." A solution exists even when it isn't obvious. If you can't find one, "cast your net farther."
- **Use the person who triggered the event.** "Oftentimes, the person with the most state is the one whose actions somehow triggered the event. Utilize that person."
- **Learn after mitigation.** Once the emergency is mitigated, set aside time to clean up, write up the incident, and learn from it ("Learn from the Past. Don't Repeat It.").
- **Test failures on purpose** ("Test-Induced Emergency", ref [Kri12]). SREs break systems deliberately, watch how they fail, and fix the weaknesses and hidden dependencies they find.

## Concrete numbers and timelines (as written)

There are no formulas, thresholds or tables in this chapter. The numbers are case-study timelines.

**Test-induced (hidden MySQL dependency):**
- The plan was to "block all access to just one database out of a hundred."
- Dependent services reported user impact "within minutes."
- "Within an hour of the original decision, all access was fully restored."

**Change-induced (global config push to abuse-protection infrastructure, on a Friday):**
- Alerts fired "within seconds."
- The push engineer rolled back "within five minutes of that first configuration push."
- On-calls declared an incident "within 10 minutes of the first push."
- Some services "didn't fully recover for up to an hour."
- Every location was "essentially offline for a few minutes."

**Process-induced (turndown automation sent machines to Diskerase):**
- Two consecutive turndown requests for the same installation; the second one hit a bug.
- "Within an hour, all traffic had been diverted to other locations". The outage was officially over, though users may have seen elevated latency.
- The first installation was rebuilt "within three hours of the initial outage."
- "Within three days, the vast majority of capacity was back online", and stragglers were recovered "over the next month or two."
- A regression "prevented the infrastructure from running more than two setup tasks per worker machine." It could not handle "the simultaneous setup of thousands of machines."
- Turndown plus secure wipe of many small installations took "less than an hour."
- Incident protocols "had matured considerably in the year since the first outage described in this chapter."

## Recommended practices and procedures

- **Abort the test at the first sign of unexpected impact.** In case 1, SRE assumed the test was responsible and aborted immediately.
- **Work in parallel.** Restore access with an already-tested approach while also engaging developers to fix the root flaw. Some teams reconfigured their systems to avoid the broken dependency.
- **Test rollback procedures beforehand.** "We now require thorough testing of rollback procedures before such large-scale tests." The untested rollback failed and made the outage longer.
- **Spread incident process changes widely.** The case 1 process was only a few weeks old and "hadn't been thoroughly disseminated." SRE keeps refining and testing its incident tools and makes sure procedure updates are communicated.
- **Canary every change thoroughly, "regardless of the perceived risk"** (case 2). A change judged low-risk got a less strict canary and hit an untested combination of keyword and feature.
- **Keep out-of-band communications and alternative access**, such as "panic rooms" with backup production access and command-line tools that can roll back changes when other interfaces are down. Use them regularly. Engineers "needed to be more familiar with the tools and to test them more routinely."
- **Use rate-limiting and throttling as defense in depth.** Rate-limiting full updates to new clients may have slowed the crash-loop.
- **Stop the automation** (case 3). On-calls disabled all team automation, then froze further automation and production maintenance.
- **Drain traffic** away from broken locations toward large installations sized for full load. Treat congested network links as the top priority.
- **Restore monitoring** quickly (by reverting the monitoring teardown) so you can see how much damage there is.
- **Recover with a structured manual process.** Hand off across time zones (US to Europe). Split the team into three parts, one per reinstall step. Raise the QoS of install traffic and restart stuck machines automatically.
- **Keep a history of outages.** Publish and organize postmortems company-wide. "Hold yourself and others accountable" for their action items. Look for fixes that are strategic as well as tactical.
- **Run proactive tests** while your best people are present and watching, not at "2 a.m. Saturday morning."

## Anti-patterns Google warns against

- Running large tests without tested rollbacks, or with a narrow understanding of dependencies. The case 1 test was "thoroughly reviewed and thought to be well scoped" and still caused an outage.
- Ignoring or not knowing the incident response process.
- Matching canary rigor to how risky a change *seems*.
- Pushing configuration globally all at once. The case 2 change went out everywhere at the same time.
- Noisy alerting during a large outage. Alerts "fired repeatedly and constantly, overwhelming the on-calls and spamming regular and emergency communication channels."
- Relying on your own production stack for debugging and communication. Much of it sat "behind jobs that were crash-looping."
- Relying on luck. The fast rollback in case 2 happened only because the push engineer was watching real-time channels, which is "not a normal part of the release process."
- Automation without sanity checks. An empty response was passed through as an empty filter, which meant "Diskerase all machines." "Yes, sometimes zero does mean all."
- Recovery infrastructure that can't handle large-scale load: TFTP at the lowest QoS, poorly tuned timeouts, and BIOS halt or reboot loops.

## Templates and checklists

The chapter has no formal template. The closest things are:

**Case-study structure:** Details, then Response, then Findings (What went well / What we learned).

**"What if…?" prompts** ("Ask the Big, Even Improbable, Questions"):
- What if the building power fails?
- What if the network equipment racks are standing in two feet of water?
- What if the primary datacenter suddenly goes dark?
- What if someone compromises your web server?
- What do you do? Who do you call? Who will write the check?
- Do you have a plan? Do you know how you, and your systems, will react?
- Could you minimize the impact if it happened now? Could the person sitting next to you do the same?

**Shared traits of good responses (Conclusion):** the responders didn't panic, pulled others in, studied earlier outages, built systems to handle those outages better, documented new failure modes, and tested proactively.

## Short verbatim quotes

- "Things break; that's life."
- "At the very worst, half of the Internet is down."
- "Yes, sometimes zero does mean all."
- "Reinforced the need for thorough canarying, regardless of the perceived risk."
- "When it comes to failures, theory and reality are two very different realms."
- "There is no greater test than reality."
