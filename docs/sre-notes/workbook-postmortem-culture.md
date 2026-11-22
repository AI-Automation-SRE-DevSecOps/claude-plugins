# SRE Workbook, Chapter 10: Postmortem Culture: Learning from Failure

Source: https://sre.google/workbook/postmortem-culture/
For blameless *philosophy* the chapter points to Chapter 15 of the first SRE book. This chapter covers practice: a bad-vs-good case study, incentives, and tooling.

## Core principles
- A blameless postmortem culture produces more reliable systems. Adopting it is "as much a cultural change as it is a technical one". Start small with a basic procedure and tune it; "there is no one size that fits all" (intro).
- Postmortems work when they are "written well, acted upon, and widely shared" (intro; *Reward Postmortem Outcomes*).
- Change systems, not people: "trying to change human behavior is less reliable than changing automated systems and processes" (*Key action item characteristics missing*).
- Focus on "what" went wrong, not "who" (*Why Is This Postmortem Better? > Blamelessness*).
- "The value of a postmortem is proportional to the learning it creates" (*Limited audience*).
- People act in good faith on the best information they had. Investigating where misleading information came from is worth more than assigning blame (*Failing to reinforce the culture*).

## Concrete numbers, thresholds and rules (exact)
- **Ben Treynor Sloss rule:** "all postmortems which follow a user-affecting outage must have at least one P[01] bug associated with them." He personally reviews exceptions, and there are "very few exceptions" (*Note* under *Key action item characteristics missing*). The rule asks for at least one P0 or P1 bug, not just "an action item".
- **Promptness:** the good postmortem was "written and circulated less than a week after the incident was closed" (incident Aug 11, 2014; published Aug 15, 2014). The bad one was published **four months** later (Dec 30) (*Delayed publication*, *Promptness*).
- **Case study facts:** main outage 17:10–17:50 (40 min). Residual pain lasted until Wed Aug 13, 07:46, nearly two days of extra latency. Satellites typically carry **60% of global queries**. The Traffic team spent ~48 hours with all hands on deck. Getting back to 50% of traffic from the edge took ~36 hours, and the normal balance took another 12 hours. Machines were taken at 16:38, but paging only started around 17:10.
- **Monitoring safety-check quirk:** the 25% config-delta check "did not trigger when 23% of the targets were removed, but did trigger when the same contents (29% of what remained) were readded". This caused a 30-minute delay before satellite monitoring was re-enabled.
- A similar incident three years later had a "dramatically reduced" blast radius and rate, thanks to the original action items.
- **Cadences:** FixIt weeks **twice a year**. The cross-functional postmortem-process review group meets **monthly**. Outage report covers the **past seven days** (weekly). Requiem holds thousands of postmortems dating back to **2009**.
- **Action item fields:** Type (prevent / mitigate / detect / repair / investigate), Priority (P0–P2), Owner, Tracking bug.

## Recommended practices and procedures
Writing (from *Why Is This Postmortem Better?*):
- **Clarity:** include a glossary and Background section. Group many action items by theme. The case study used five themes: Prevention/risk education, Emergency response, Monitoring/alerting, Satellite/edge provisioning, Cleanup/miscellaneous.
- **Quantify impact:** use consistent numbers across services, such as QPS lost, % of traffic, RTT, revenue ranges with error bars, and cache hit rates. Link to sources. "A well-informed estimate is better than no data at all". Include specific numbers internally.
- **Impact sections:** User, Revenue, and **Team impact** (on-call and interrupt load on other teams).
- **Root causes *and* trigger:** go deep. In the case study, a latent non-idempotent input-validation bug was triggered by a manual workflow re-run, and "run once" semantics didn't apply across workflow instances.
- **Action items:** every one gets an owner, a tracking bug, and a priority. Each needs a verifiable end state (e.g., "Add an alert when more than X% of our machines have been taken away"). Every theme should include prevent/mitigate items.
- **Ownership:** a single owner with multiple collaborators.
- **Conciseness:** summarize chat logs and system logs, and link to the raw versions.
- **Lessons Learned:** went well / went poorly / where we got lucky.
- Share by default with everyone in the company, possibly with customers too. Add machine-readable tags and metadata for analytics.

Organizational (*Organizational Incentives*):
- Leaders model blameless language and reframe blameful questions (e.g., ask "were there warning signs, and why did we dismiss them").
- Include all incident participants in writing the postmortem. Use a clear review process.
- Reward action-item *closeout*, not just writing. Reward org-wide change with peer bonuses, performance reviews and promotion. Highlight reliability gains. Hold owners up as experts. Gamify with leaderboards and burndowns.
- Share openly: org announcements and all-hands, cross-team reviews and Postmortem Reading Clubs, Wheel of Misfortune reenactments with the original Incident Commander present, a weekly outage report, and a periodic "greatest hits" report.

## Anti-patterns Google warns against (*Why Is This Postmortem Bad?*, *Respond to Postmortem Culture Failures*)
- Missing context (no glossary or background).
- Key details omitted (duration as the only number, shallow root cause, empty Recovery Efforts).
- Action items that are mostly mitigative, all the same priority, vague ("Improve", "Make better"), without tracking bugs, or aimed at training humans.
- Finger pointing at named individuals. It makes people risk-averse and motivates them to cover up facts.
- Animated language ("careless ignorance", "which is ridiculous", "!!!"). It erodes psychological safety, so provide verifiable data instead.
- Missing ownership (four owners; action items with no owner).
- Limited audience (shared only with the team).
- Delayed publication.
- Culture failures: *avoiding association*, *failing to reinforce the culture* (executive blame framed as a "safe space"), *lacking time to write postmortems*, and *repeating incidents*. For repeats, ask: are action items slow to close, is feature velocity beating reliability fixes, are the right action items being captured, is the service overdue for a refactor, are people putting Band-Aids on a deeper problem? Then bring the collaborators from each similar incident together.

## Templates and checklists (*Tools and Templates*)
- **Template header:** Owner, Shared with, Status, Incident date, Published. **Executive Summary:** Impact, Root cause. **Problem Summary:** Duration, Product(s) affected, % of product affected, User impact, Revenue impact, Detection, Resolution. Then Background (optional), Impact, Root Causes and Trigger, Timeline/Recovery Efforts, Lessons Learned, Action Items, Glossary, Appendix and Graphs. Google's template and full checklist are at https://g.co/SiteReliabilityWorkbookMaterials.
- **Checklist examples:** complete impact assessment, root-cause analysis detailed enough to drive action item planning, action items vetted and approved by the service's tech leads, postmortem shared with the wider org.
- **Tooling:** the incident tool pre-populates roles, the timeline and IRC logs, affected and root-cause services, severity, and detection mechanism. **Requiem** stores postmortems and parses their metadata. Action items are filed in a central bug tracker so their closure can be monitored. Analysis covers postmortems per month per org, mean duration, time to detect, time to resolve, and blast radius. Third-party options: PagerDuty Postmortems, Etsy's Morgue, VictorOps.

## Short verbatim quotes
- "To our users, a postmortem without subsequent action is indistinguishable from no postmortem." (Ben Treynor Sloss)
- "Let's plan for a future where we're all as stupid as we are today." (Dan Milstein)
- "Postmortems are letters you write to future team members"
- "All commands should be idempotent, or at least fail-safe on repeat invocations."
- "Even if there is no concrete data, a well-informed estimate is better than no data at all."
