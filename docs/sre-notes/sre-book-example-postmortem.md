# Appendix D — Example Postmortem (Google SRE Book)

Source: https://sre.google/sre-book/example-postmortem/
Subject: "Shakespeare Sonnet++ Postmortem (incident #465)", dated 2015-10-21. This is a filled-in, fictional, partly humorous template (DeLorean, "flux capacitor", "goat teleporter"). It is not an essay. Most of the prescriptive content is in the numbered footnotes (163–168), which explain what each section is for. Companion chapters: Ch. 15 "Postmortem Culture", Ch. 14 "Managing Incidents", Appendix C "Example Incident State Document".

## Core principles (as shown by the example)
- **Separate root cause from trigger.** Root cause here is a *latent* resource leak combined with exceptionally high load, leading to cascading failure. The trigger is the "sudden increase in traffic". The leak normally went "unnoticed" because its failure rate was low.
- **Impact means effect on users and revenue** (fn 163), stated in numbers.
- **Root-cause analysis should look for contributing factors**, "often helpful to use a technique such as the 5 Whys [Ohn88]" (fn 164).
- **Action items need judgment.** Avoid knee-jerk fixes and prefer mechanisms that catch problems earlier, such as unit tests, over bolting on more monitoring (fn 165).
- **Error budget has consequences.** An exhausted budget leads to a production freeze unless an exception is granted.
- **Record luck and near misses** as their own section (fn 166).

## Concrete numbers, thresholds and facts (exact)
- Summary: "Shakespeare Search down for **66 minutes**". The outage ran from 14:54 (OUTAGE BEGINS) to 16:00 (OUTAGE ENDS) UTC.
- Impact: "Estimated **1.21B queries lost**, no revenue impact."
- Traffic rose **88x** at 14:53, after a /r/shakespeare post.
- HTTP 500 rate reached **~100%**.
- Error budget exceeded "**by several orders of magnitude**".
- Capacity: first a **2x** instance increase (15:28). Then an **order-of-magnitude error** turned up in the whiteboard calculations (15:34). After that came an additional global **5x** increase, "to a total of **10x** initial capacity".
- Traffic was ramped back to the non-sacrificial clusters in steps: **1% → 10% → 30% → 50% → all** (15:41, 15:45, 15:50, 15:55, 16:00).
- Incident exit criterion: "**30 minutes' nominal performance**". The incident ended at 16:30.
- Freeze: "Freeze production until **2015-11-20** due to error budget exhaustion". That is about one month after the incident.
- Key timestamps: news 14:51. Pager storm (`ManyHttp500s`) 14:55. **INCIDENT BEGINS 15:01**, when docbrown declares #465 and names jennifer incident commander. Smoking gun (file descriptor exhaustion) 15:18. **OUTAGE MITIGATED 15:36**. Outage began 7 minutes before the incident was declared. The incident (15:01–16:30) is a different span from the outage (14:54–16:00).

## Template / checklist (header fields, in order)
| Field | Content in example / guidance |
|---|---|
| Title + incident # | "Shakespeare Sonnet++ Postmortem (incident #465)" |
| Date | 2015-10-21 |
| Authors | jennifer, martym, agoogler |
| Status | "Complete, action items in progress" |
| Summary | One line: what failed, for how long, and the context |
| Impact | Effect on users, revenue, etc. (fn 163) |
| Root Causes | Circumstances in which it happened; 5 Whys (fn 164) |
| Trigger | "Latent bug triggered by sudden increase in traffic" |
| Resolution | Mitigations + permanent fixes, and what is still being maintained |
| Detection | "Borgmon detected high level of HTTP 500s and paged on-call" |
| Action Items | Table: Action Item / Type / Owner / Bug (+ DONE/TODO) |
| Lessons Learned | What went well / What went wrong / Where we got lucky |
| Timeline | "Screenplay", seeded from the incident management doc (fn 167) |
| Supporting information | Links, logs, screenshots, graphs, IRC/IM logs (fn 168) |

**Action item types used:** `mitigate`, `prevent`, `process`, `other`. Every item has a single named owner. Most have a bug ID (e.g., Bug 5554823). Status is DONE or TODO. Of 9 items: 6 prevent, 1 mitigate, 1 process, 1 other. 3 are DONE.
Examples: update the playbook for cascading failure (mitigate). Schedule a cascading-failure test during the next **DiRT** (process). Add load shedding. Build regression tests for **queries of death**. Plug the file descriptor leak. Investigate running the index MR/fusion continuously.

**Timeline conventions:** a single timezone ("all times UTC"). Milestone markers in caps: OUTAGE BEGINS, INCIDENT BEGINS, OUTAGE MITIGATED, OUTAGE ENDS, INCIDENT ENDS. Actors are named. Mistakes are recorded plainly (the order-of-magnitude error).

## Recommended practices & procedures (from the incident response)
- Declare the incident early, open a coordination channel (#shakespeare), name an incident commander, and notify the incidents mailing list.
- Bring in the dev team for code-level diagnosis.
- For cascading failure, use a **sacrificial cluster**. Send all traffic to one cluster (USA-2) so the other clusters can scale up without tasks dying straight away. Then re-add traffic gradually by percentage.
- If a capacity increase fails, revert to the safe state, recompute, and scale again.
- Treat the fix as two parts: an immediate mitigation (capacity plus an updated index) and a permanent fix (the leak fix, load shedding, regression tests).
- Keep extra capacity "until surge in public interest in new sonnet passes".
- Use a defined exit criterion before you end the incident.

## Anti-patterns warned against
- **"Knee-jerk" action items** that are "too extreme or costly". Re-scope them in a larger context (fn 165).
- **Over-optimizing for one specific issue**, for example adding specific monitoring or alerting when unit tests would catch it earlier (fn 165).
- **Being out of practice** with a failure mode ("What went wrong"). The fix is a DiRT exercise.
- **Treating luck as success.** Things that only worked by chance go under "Where we got lucky", not "What went well".

## Short verbatim quotes
- "Under normal circumstances, the rate of task failures due to resource leaks is low enough to be unnoticed."
- "Impact is the effect on users, revenue, etc."
- "There's a risk of over-optimizing for a particular issue"
- "This section is really for near misses"
- "A 'screenplay' of the incident"
- "reached exit criterion of 30 minutes' nominal performance"
- "We're out of practice in responding to cascading failure"
