# SRE Book, Chapter 31: Communication and Collaboration in SRE

Source: https://sre.google/sre-book/communication-and-collaboration/
Lead author: Niall Murphy.

## Core principles

- **SRE is not command-and-control** (intro). Service/infra SRE teams answer to "at least two masters": the product dev team they work with, and SRE as a whole, which is where they report. Shared culture produces "strongly homogeneous approaches to problems" on purpose.
- **Team interface as an API** (intro). Information has to flow reliably between teams. Design the SRE team's interface like an API, because a bad one "can be painful to correct later on."
- **A full skill set, not just ownership.** The promise of SRE is that a reliability org with the same skills as product dev "will improve things measurably". Just putting someone in charge of reliability without those skills "is not enough."
- **Production meetings are about services, not people** (Communications: Production Meetings). Everyone should leave with the *same* idea of what is going on. The meeting ties operational performance back to design, configuration, and implementation, which the chapter calls "an immensely powerful feedback loop."
- **Specialization cuts both ways** (Collaboration within SRE). It lowers cognitive load and builds mastery, but it also creates silos. Write a crisp team charter that says what the team will and, more importantly, *won't* support.
- **Get involved early** (Collaboration Outside SRE). Collaboration with product dev works best in the design phase, "ideally before any line of code has been committed", because architecture is hard or impossible to retrofit. This work is tracked with OKRs.

## Concrete numbers and facts (exact)

| Item | Value as written |
|---|---|
| Production meeting frequency | "usually happen weekly" |
| Production meeting length | "somewhere between 30 and 60 minutes" |
| Shorter than 30 min | you are "cutting something unnecessarily short, or you should probably be growing your service portfolio" |
| Longer than 60 min | you are "mired in the detail", or you should "shard the team or service set" |
| Nonpaging events bucket | 3 items |
| Formal SRE team roles | tech lead (TL), manager (SRM), project manager (PM/TPM/PgM) |
| Manager-only duties | 2: performance management, and being the "general catchall" |
| Viceroy timeline | duplicate team efforts over 12–18 months; Viceroy born mid-2012; prototypes integrated with Consoles++ early 2014; declared the SRE-wide solution by end of 2014 |
| Viceroy contributor stints | "between one and three months" |
| DFP indexing system | ~1,000 CPUs, 8 TB RAM, indexing 100 TB of data per day; migrated from MySQL to F1 |
| DFP PD/SRE sync | weekly meetings |

Note: Viceroy was *recommended*, not mandated. Teams were told to use it "instead of writing another monitoring console."

## Recommended practices and procedures

**Running a production meeting** (Production Meetings, Agenda, Attendance)
- Have a chair, and **rotate the role**. Shared ownership is worth some temporary suboptimality, and chairing builds incident-coordination skills.
- When two teams meet over video and one is much larger, **put the chair on the smaller side** by default. Footnote 147 says the larger side tends to talk over the smaller one.
- **Attendance is compulsory** for every member of the SRE team, especially when the team spans countries or time zones. Major stakeholders and partner product dev teams should also attend. Splitting the meeting so SRE-only topics come first is fine.
- If you can't invite your product dev partners, that relationship needs fixing. Start by inviting one representative, or use a trusted intermediary.
- Too many attendees: send one PD representative or a subset of them, have PD commit to reading the minutes, or let busy people give feedback in advance.
- Keep a shared, real-time collaborative agenda doc at a well-known address that anyone in engineering can open. Pre-populate it bottom-up and prepare it in parallel, in advance.

**Cross-site projects** (Viceroy: Recommendations)
- Only go cross-site when you have to. The cost is "higher latency for actions and more communication". The payoff, if the mechanics are right, is "much higher throughput". Single-site projects have their own cost: nobody outside the site knows what you're doing.
- Make sure contributors are actually committed and have a specific goal.
- Divide and conquer: split the work into reasonably sized components, each owned by a small group, ideally on one site, with clear deliverables and deadlines. Don't let Conway's law distort the shape of the software.
- Give each component a goal framed as functionality to deliver. A component is done only once it is integrated and used in the main project.
- Every component gets a design doc and a team review. "Writing things down" is the main way to offset distance.
- Standards: argue each debate out fully but under a strict time limit, then pick, document, and move on. If you can't agree, use an arbitrator everyone respects.
- Meet in person. Project leaders should meet the team, and hold a team summit if budget allows, at a neutral location when neutrality matters.
- Scale project management to the project's current size. Keep overhead low early. Full PM becomes "necessary" once the project has grown enough.

**SRE/PD migration pattern** (Case Study: Migrating DFP to F1)
- SRE drives infrastructure design. PD owns business logic (BL). Two SREs wrote the design doc, both teams reviewed it, and the infra/BL interfaces were defined early so each team could work independently.
- Validate in a production-like test environment until the new output matches the old *perfectly*. Prepare production in parallel (monitoring, on-call training, a release process with validation), then roll out together.

## Anti-patterns Google warns against

- Meetings that track individuals' status instead of services.
- Change meetings built to *stop* change. Google "default[s] to enabling change" and tracks it.
- Keeping unactionable pages or alerts that need no attention. Remove them as noise.
- Singleton projects, which "fail unless the person is particularly gifted or the problem is straightforward."
- Relying only on remote contact: "you decay into just being an email address until you turn up in the flesh again."
- Duplicated tooling, the "smoldering, abandoned hulks of monitoring frameworks". This is driven by rewarding each team for its own solution, the difficulty of working across team boundaries, and shared infra that is "closer to a toolkit than a product."
- Casual contributions that dilute ownership: features go unsupported after the contributor leaves.
- Making decisions locally without the remote owners, which "can lead to conflict."

## Template: default production meeting agenda (Agenda; see Appendix F)

1. **Upcoming production changes.** Start time, duration, expected effect. This gives "near-term horizon visibility."
2. **Metrics.** Core SLO metrics, latency and CPU trends over time to learn the "performance envelope". Optionally resource usage and efficiency.
3. **Outages.** Problems of roughly postmortem size.
4. **Paging events.** List of pages, who was paged, what happened. Ask "should that alert have paged in the way it did, and should it have paged at all?"
5. **Nonpaging events.**
   - (a) Should have paged but didn't: fix the monitoring.
   - (b) Not pageable but needs attention, e.g. low-impact data corruption or non-user-facing slowness. Track reactive ops work here too.
   - (c) Not pageable and needs no attention: remove the alert.
6. **Prior action items.** Assign owners and track progress. Keep a catchall item.

## Short verbatim quotes

- "The goal is for everyone to leave the meeting with an idea of what's going on—the same idea."
- "Connecting the performance of the service with design decisions in a regular meeting is an immensely powerful feedback loop."
- "Writing things down is one of the major techniques you have to offset physical and/or logical distance—use it."
- "simply having someone in charge of reliability, without also having the complete skill set, is not enough."
