# SRE Workbook, Chapter 9: Incident Response

Source: https://sre.google/workbook/incident-response/. Authors: Jennifer Mace, Jelena Oertel, Stephen Thorne, Arup Chakrabarti (PagerDuty), with Jian Ma and Jessie Yang. (The page's HTML `<title>` reads "Root Cause Analysis for Probing Incident", which looks like a site metadata error. The body heading is "Chapter 9 - Incident Response".)

## Core principles

**Definitions (intro):** *Resolving* an incident means "mitigating the impact and/or restoring the service to its previous condition." *Managing* an incident means coordinating responders efficiently and making sure communication flows both among responders and to people who are interested.

**Basic principles of incident response (intro, exact list):**
1. Maintain a clear line of command.
2. Designate clearly defined roles.
3. Keep a working record of debugging and mitigation as you go.
4. Declare incidents early and often.

**Incident Command System (ICS):** Google's system, IMAG (Incident Management At Google), is based on ICS. Firefighters created ICS in **1968** to manage wildfires.

**The "three Cs" (3Cs):** **Coordinate** the response effort; **Communicate** between responders, within the organization, and to the outside world; maintain **Control** over the response. When incident response goes wrong, "the culprit is likely in one of these areas."

**Main Roles in Incident Response:**
- **Incident Commander (IC):** usually the person who declares the incident. The IC commands and coordinates, delegates roles, and **by default holds every role not yet delegated**. The IC can hand off command and take the OL role, or assign OL to someone else.
- **Communications Lead (CL):** the public face of the response team. Sends periodic updates to responders and stakeholders and handles inquiries. In a small incident the IC can absorb the CL role again.
- **Operations Lead (OL):** applies operational tools to mitigate or resolve the incident.
- In IMAG the roles form a hierarchy: **the CL and OL report to the IC**. The CL and OL can each lead their own sub-teams, which grow or shrink as needed.

**Mitigation first (Case Study 2 review):** "first responders must *prioritize mitigation above all else*, or time to resolution suffers." The priority order is:
1. Assess the impact.
2. Mitigate the impact.
3. Perform root-cause analysis.
4. After the incident, fix the cause and write a postmortem.

## Concrete numbers and facts (as written)

- **Footnote 1:** "When three or more people work on an incident," start a collaborative document that lists working theories, eliminated causes, and useful debugging information such as error logs and suspect graphs.
- **Case Study 1 (Google Home):** the v1.88 bug fetched speaker-recognition files "50 times more often than expected." The data was pinged every 30 minutes, which is 48 times a day, instead of once a day. The rollout was paused at 25% and restarted, reached 50% on May 31, and reached 100% on Saturday, June 3. During the weekend of June 3, 2017, users lost half of their requests. The quota was raised several times. The final increase came at 3:33 p.m. Sunday and ended the impact. No incident was ever declared.
- **Case Study 2 (GKE CreateCluster):** paged at 6:41 a.m. with prober failures above 60% in two zones. The incident was declared at 7:06 a.m. A formal structure was set up "two hours after the first page." The outage lasted 6 hours 40 minutes. 41 unique IRC users took part, the IRC logs ran to 26,000 words, 7 IMAG task forces were spun up (up to 4 at the same time), on-calls came from 6 teams, and the postmortem had 28 action items. A generic rollback after the possible cause was found (9:56 a.m.) would have mitigated the incident "by 10 a.m." The actual fix came at 11:59 a.m. (corrupt image purged from the GCR mirror), and errors reached 0% by 12:11 p.m.
- **Case Study 3 (Belgium power):** in mid-2015, lightning struck 4 times within 2 minutes. Disk-tray UPSs failed to switch over on strikes 3 and 4. Only **0.000001%** of data from running GCE machines was lost.
- **Case Study 4 (PagerDuty NTP drift, Oct 6, 2017):** the incident lasted more than 10 hours with minimal customer impact. It was first alerted at 7:53 p.m. and declared a major incident at 9:49 p.m. The **on-call engineers and the IC rotated every four hours.** It ended at 6:13 a.m. At the time PagerDuty had over 400 employees. Failure Friday runs **weekly**.

## Recommended practices and procedures

- **Declare early (CS1 review).** "Our experience shows that managed incidents are resolved faster." Declaring early prevents miscommunication, speeds up root-cause identification and resolution, and brings the relevant teams in sooner.
- **Centralize communication.** Use a physical or virtual "war room" (IRC, Hangout). When discussion gets too dense, move detailed debugging into a shared document and keep the chat channel as "the hub for decision making" (CS2).
- **Generic mitigations (CS2 review).** These are blunt but fast actions, such as rolling back a release or steering load balancers away from a region. You need to know the *location* of the root cause, not its details. Build mitigation tools before incidents happen, and mine postmortems for ideas.
- **Pause the rollout until the root cause is known** after the first mitigation (CS1). Roll out only on business days, or provide paid on-call coverage.
- **Hand off command to the more experienced person** when that helps (CS2). The IC stays with the team that has the best view of customer impact (CS3).
- **PagerDuty's split:** make coordination decisions on the conference call, record the outcomes in Slack (#incident-war-room). A scribe captures actions, owners, and timestamps, and every call is recorded.
- **Incident Response Training:** you don't have to adopt full ICS. Choose parts such as: tell on-calls they can delegate and escalate, encourage a mitigation-first response, and define the IC, CL, and OL roles. Use hands-on exercises built from past incidents.
- **Prepare Beforehand:** agree on a communication channel ahead of time, ideally one the team already knows. Send regular status updates, and explicitly announce when the response is called off. Plan how blog posts and press releases get approved (at Google, with PR). Prepare **"two or three ready-to-use templates"**, keep a contact list, and set criteria for what counts as an incident, drawn from past outages and known high-risk areas.
- **Drills:** DiRT (company-wide, a controlled emergency with no customer impact), Wheel of Misfortune, treating minor problems as major ones, Failure Friday, and "Keep Talking and Nobody Explodes." Run drills periodically with real tools, and write a follow-up report on what went well, what didn't, and what could improve.

## Anti-patterns Google warns against

- Not declaring an incident and troubleshooting by "normal" methods through the bug tracker. In CS1 this went on even after users had been reporting the problem for days.
- Relying on heroics: "successful incident management shouldn't rely on heroic efforts of individuals."
- Resuming a rollout after mitigation but before the root cause is known. Rolling out on weekends.
- An IC who never sets up formal structure, so first responders "pursued their own investigations without coordination" (CS2).
- Having no generic mitigations ready, and building bespoke fixes in the middle of an incident (a rebuild of about 1 hour that had to be restarted).
- Silence. Without updates, people assume nothing is being done, and if the response isn't called off they assume the incident is still ongoing.
- Choosing the communication channel during the incident.

## Templates and checklists

- **Starter checklist (Incident Response Training):** delegation and escalation allowed; mitigation first; IC, CL, and OL roles defined.
- **Preparation checklist (Prepare Beforehand):** communication channel, plan for keeping the audience informed, 2 to 3 communication templates, contact list, incident criteria.
- **Active-incident order:** assess, then mitigate, then root-cause, then fix and postmortem.
- **Working-document contents (fn. 1):** working theories, eliminated causes, error logs, suspect graphs.

## Short verbatim quotes

- "By default, the IC assumes all roles that have not been delegated yet."
- "To mitigate an incident, you don't have to fully understand the details—you only need to know the location of the root cause."
- "Ultimately, customers do not care whether or not you fully understand what caused an outage."
- "By the time a pager alerts you to a problem, it's too late to think about how to manage the incident."
- "The right time to create general-purpose mitigation tools is before an incident occurs."
