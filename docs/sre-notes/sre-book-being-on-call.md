# SRE Book, Chapter 11: Being On-Call

Source: https://sre.google/sre-book/being-on-call/ (written by Andrea Spadaccini, edited by Kavita Guliani; an earlier version appeared in *;login:*, Oct 2015, vol. 40, no. 5)

Sections: Introduction · Life of an On-Call Engineer · Balanced On-Call (Balance in Quantity, Balance in Quality, Compensation) · Feeling Safe · Avoiding Inappropriate Operational Load (Operational Overload, A Treacherous Enemy: Operational Underload) · Conclusions

## Core principles

- **Engineering is what SRE is for** (Introduction, Balance in Quantity). Purely operational work is capped at 50%. At least 50% of SRE time goes to engineering projects.
- **On-call is measured on two axes** (Balanced On-Call). *Quantity* is the percent of time spent on on-call duties. *Quality* is the number of incidents per on-call shift. SRE managers are responsible for keeping both balanced and sustainable.
- **Response time follows from the SLO** (Life of an On-Call Engineer). Paging response times are agreed between the team and the business system owners, and they come from the availability target.
- **Deliberate thinking beats intuition during incidents** (Feeling Safe). Citing Kahneman [Kah11], the chapter contrasts "intuitive, automatic, and rapid action" with "rational, focused, and deliberate cognitive functions". The second gives better results with complex systems, so reducing on-call stress is a reliability concern, not only a welfare concern.
- **Blameless, event-focused postmortems.** Google looks at systems, not individuals. Finding automation opportunities is "one of the best ways to prevent human errors."
- **Operational load can be too high or too low.** Both overload and underload are harmful.

## Concrete numbers, thresholds, formulas

| Item | Value as written |
|---|---|
| Max operational (non-project) work | 50% |
| Min engineering/project work | 50% |
| Max time on-call | no more than 25% (of total time; it is part of the 50% operational share) |
| Other operational, non-project work | up to another 25% |
| Paging response time, user-facing / highly time-critical | typically 5 minutes |
| Paging response time, less time-sensitive systems | typically 30 minutes |
| 99.99% quarterly availability | ~13 minutes allowed quarterly downtime, so reaction time "strictly speaking, 13 minutes" |
| Min engineers, single-site 24/7 team (primary + secondary, week-long shifts) | **8** (each on-call, primary or secondary, one week every month) |
| Min size per site, dual-site team | **6** at each site |
| Average time to handle one incident (RCA, remediation, postmortem, bug fixes) | **6 hours** |
| Max incidents | **2 per 12-hour on-call shift** |
| Desired paging distribution | "very flat over time, with a likely median value of 0" |
| Warning sign | median incidents/day > 1 (a component paging every day) |
| Example overload goals | "number of daily tickets < 5, paging events per shift < 2" |
| Target alert/incident ratio | approach 1:1 |
| Min on-call exposure (underload) | every engineer on-call "at least once or twice a quarter" |

Derivations:
- 8 engineers: 2 people on-call at all times, week-long shifts, so each engineer covers 1 week in 4. That is 25%.
- 2 incidents per shift: 12 h ÷ 6 h per incident = 2.

**Definition of an incident** (Balance in Quality): "a sequence of events and alerts that are related to the same root cause and would be discussed as part of the same postmortem." It is not the same thing as a single page or alert.

## Recommended practices and procedures

- **Primary and secondary rotations** (Life of an On-Call Engineer). How duties are split varies by team. The secondary can be a fall-through for pages the primary misses, or the primary can handle only pages while the secondary handles non-urgent production work. Two related teams can serve as secondary for each other, which removes the need for a dedicated secondary rotation.
- **After acknowledging a page**: triage, work toward resolution, and involve teammates and escalate as needed. Pages take priority over almost everything, including project work. Non-paging events (lower-priority alerts, releases) are handled during business hours.
- **Prefer multi-site "follow the sun"** once a service justifies growing a single-site team. This avoids night shifts, which harm health [Dur05], and keeps rotations small enough that engineers stay in touch with production. The cost is communication and coordination overhead, so weigh the trade-offs, the system's importance, and its workload.
- **Compensation**: Google offers time-off-in-lieu or cash, capped at some proportion of salary. The cap in practice limits how much on-call work any one person takes on, which prevents burnout and protects project time.
- **On-call safety resources**: clear escalation paths, well-defined incident-management procedures, and a blameless postmortem culture. Developer teams usually run their own 24/7 rotation and can always be escalated to.
- **Use a formal incident protocol** (from *Managing Incidents*) when multiple teams are involved, or when you cannot yet estimate an upper bound on how long the incident will last. Google backs this with a web tool that automates role handoffs and status communication.
- **Write postmortems after significant incidents**, with a full timeline of events.
- **Fixing overload**: put concrete objectives in quarterly planning. Temporarily loan an experienced SRE to the team (Ch. 30). Make the symptoms measurable.
- **Alert hygiene**: page on symptoms that threaten SLOs, and make every page actionable. Group related alerts to limit fan-out, and silence duplicate or uninformative alerts during an incident.
- **When developer changes cause noise**: set shared goals with the developers. In extreme cases, "give back the pager", or route some or all pages to the developer on-call, usually as a temporary measure until the service can be on-boarded again.
- **Fixing underload**: size teams so everyone is on-call at least once or twice a quarter. Run "Wheel of Misfortune" exercises and the annual company-wide DiRT (Disaster Recovery Training) drills [Kri12].

## Anti-patterns Google warns against

- Rotations so large that engineers lose touch with production (operational underload). This leads to over- or under-confidence and knowledge gaps that only show up during an incident.
- A component or issue that pages every day (median > 1/day).
- Low-priority alerts that interrupt hourly or more often. They cause fatigue, and then serious alerts get less attention than they need.
- Non-actionable pages, and pages that are not tied to SLO-threatening symptoms.
- Alert fan-out: many alerts for one incident.
- Confirmation bias and heuristic abuse under stress. The example given is assuming the fourth page of the week has the same cause as the previous three.
- Habitual, unconsidered quick reactions, which "can be disastrous."
- Blaming individuals instead of analyzing events.
- Letting SRE absorb excessive load during architectural fixes that take several quarters.

## Templates and checklists

The chapter gives no formal template. These checklists follow from it:

**On-call sustainability check**
- [ ] On-call ≤ 25%, operational ≤ 50%
- [ ] Headcount ≥ 8 (single site) or ≥ 6 per site (dual site)
- [ ] ≤ 2 incidents per 12-hour shift; median ≈ 0
- [ ] Response target derived from the SLO
- [ ] Everyone on-call ≥ 1–2×/quarter
- [ ] Alerts actionable, SLO-based, ≈ 1:1 per incident
- [ ] Escalation paths, incident protocol, blameless postmortems
- [ ] Capped compensation

## Short verbatim quotes

- "We cap the amount of time SREs spend on purely operational work at 50%" (Introduction)
- "no more than 25% can be spent on-call, leaving up to another 25% on other types of operational, nonproject work." (Balance in Quantity)
- "It follows that the maximum number of incidents per day is 2 per 12-hour on-call shift." (Balance in Quality)
- "All paging alerts should also be actionable." (Operational Overload)
- "Noisy alerts that systematically generate more than one alert per incident should be tweaked to approach a 1:1 alert/incident ratio." (Operational Overload)
- "An operational underload is undesirable for an SRE team." (A Treacherous Enemy: Operational Underload)
