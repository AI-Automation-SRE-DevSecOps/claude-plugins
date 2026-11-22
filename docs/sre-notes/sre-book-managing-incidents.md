# Chapter 14 — Managing Incidents (Google SRE Book)

Source: https://sre.google/sre-book/managing-incidents/
Author: Andrew Stribblehill; editor: Kavita Guliani. An earlier version appeared in *;login:* (April 2015, vol. 40, no. 2). Copyright 2017 Google, CC BY-NC-ND 4.0.

Chapter structure: Unmanaged Incidents → The Anatomy of an Unmanaged Incident → Elements of Incident Management Process → A Managed Incident → When to Declare an Incident → In Summary → Best Practices for Incident Management.

## Core principles

- **Prepare before the incident.** "If you haven't gamed out your response to potential incidents in advance, principled incident management can go out the window in real-life situations." (intro)
- **Based on the Incident Command System (ICS)** (FEMA's National Incident Management System is cited in footnote 79), "known for its clarity and scalability." (Elements of Incident Management Process)
- **Process exists to channel enthusiasm.** Incident management skills "exist to channel the energies of enthusiastic individuals." In the failure scenario, "everybody … was doing their job, as they saw it."
- **Recursive separation of responsibilities.** Each person knows their role and stays in it. Clear separation gives people *more* autonomy, because they don't have to second-guess colleagues. It is recursive: an overloaded member asks the **planning lead** for more staff and delegates, which may mean creating **subincidents**. A role lead can also hand system components to colleagues, who report high-level information back up.
- **Outcome claimed** (In Summary): planning in advance, structuring the plan to scale, and using it regularly reduced **mean time to recovery** and gave staff "a less stressful way to work on emergent problems."

## Roles (Recursive Separation of Responsibilities)

| Role | Responsibilities as written |
|---|---|
| **Incident Command** | Holds the high-level state. Structures the task force and assigns responsibilities by need and priority. *De facto* holds every position not yet delegated. Removes roadblocks for Ops. |
| **Operational Work (Ops lead)** | Works with the IC and applies operational tools. "The operations team should be the only group modifying the system during an incident." |
| **Communication** | The task force's public face. Issues periodic updates to the team and stakeholders ("usually via email"). May also keep the incident document accurate and current. |
| **Planning** | Supports Ops on longer-term issues: filing bugs, ordering dinner, arranging handoffs, and tracking how the system has diverged from normal so it can be reverted after the incident. |

Note: the chapter's main text says the **incident commander's** most important job is keeping the living incident document. The Communication role "may extend to" maintaining it.

## Concrete numbers / thresholds

The chapter is light on numbers. Here are the exact ones:

- **Declaration criteria** (When to Declare an Incident). The event is an incident "if any of the following is true" (an OR, not an AND):
  1. "Do you need to involve a second team in fixing the problem?"
  2. "Is the outage visible to customers?"
  3. "Is the issue unsolved even after an hour's concentrated analysis?"
  The author calls these "broad guidelines" his own team follows ("My team follows…"). They are not a Google-wide mandate.
- **Scenario details** (illustrative only): 5 datacenters, page at 2 p.m. Thursday. In the managed version, replacement staff are sought at 5 p.m., a phone conference is held at 5:45, and the handoff to the sister office happens at 6 p.m.
- The chapter defines no formulas, SLO thresholds, or severity levels.

## Recommended practices & procedures

- **Recognized command post** (A Recognized Command Post). Interested parties must know where to reach the IC. That can be a central "War Room", or people at their desks following email and IRC. Google found IRC "a huge boon": it is reliable, it keeps a log, it suits geographically distributed teams, and bots log incident traffic (for postmortems) and alerts to the channel.
- **Live incident state document** (Live Incident State Document). It can live in a wiki but should ideally support several concurrent editors. Most teams use Google Docs, but Google Docs SRE use Google Sites, so they don't depend on the system they are fixing. It "can be messy, but must be functional." Use a template, keep the most important information at the top, and retain it for postmortem and meta-analysis. See Appendix C, "Example Incident State Document."
- **Clear, live handoff** (Clear, Live Handoff). Hand off IC explicitly at the end of the working day, by phone or video if the next IC is at another site. The outgoing IC says "You're now the incident commander, okay?" and stays on the call until handoff is firmly acknowledged. Then tell everyone working the incident who leads now.
- **Managed-incident flow** (A Managed Incident):
  - The on-call engineer hands command to a colleague ("Can you take command?") and keeps doing Ops.
  - The IC emails a prearranged mailing list, records the impact assessment in the live doc, and follows up on the thread when the situation escalates. This keeps VPs informed "without bogging them down in minutiae."
  - The IC asks external communications to draft user messaging.
  - The IC loops in the developer on-call after Ops approves.
  - Helpers prioritize tasks delegated by the Ops lead and keep the Ops lead informed of any extra actions. They get up to speed by reading the incident doc.
  - Failed fix attempts are posted to IRC and copied into the doc.
- **Declare early.** It is better to declare early and close quickly than to spin up the framework hours into a growing problem. Set clear declaration conditions.
- **Keep skills fresh.** Proficiency "atrophies quickly." Use the framework for ordinary operational changes that span time zones or teams. Include it in disaster-recovery testing (DiRT, [Kri12]). Role-play past, already-solved on-call issues.

## Anti-patterns (The Anatomy of an Unmanaged Incident)

1. **Sharp focus on the technical problem.** The responder is so absorbed in fixing that nobody thinks about mitigation or the bigger picture.
2. **Poor communication.** Nobody knows what coworkers are doing, leadership and customers are left uninformed, and available helpers aren't used.
3. **Freelancing.** Uncoordinated, well-meant changes. In the scenario, Malcolm's CPU-affinity change, pushed straight to production, made the restarted servers die.

Also shown in the scenario: a solo on-call who never hands off command, ad hoc late-night calls to developers, onlookers who are "just looking" without assigned roles, and executives demanding ETAs straight from the responder.

## Checklist — Best Practices for Incident Management (verbatim headings)

- **Prioritize.** Stop the bleeding, restore service, and preserve the evidence for root-causing.
- **Prepare.** Develop and document procedures in advance, in consultation with incident participants.
- **Trust.** Give full autonomy within the assigned role to all incident participants.
- **Introspect.** Watch your emotional state. If you feel panicky or overwhelmed, ask for more support.
- **Consider alternatives.** Periodically re-evaluate whether to continue on your current path or take another tack.
- **Practice.** Use the process routinely so it becomes second nature.
- **Change it around.** Rotate roles so every team member becomes familiar with each one.

## Short verbatim quotes

- "The operations team should be the only group modifying the system during an incident."
- "De facto, the commander holds all positions that they have not delegated."
- "The incident commander's most important responsibility is to keep a living incident document."
- "You're now the incident commander, okay?"
- "Incident management proficiency atrophies quickly when it's not in constant use."
- "Depending on the software you are trying to fix as part of your incident management system is unlikely to end well."
