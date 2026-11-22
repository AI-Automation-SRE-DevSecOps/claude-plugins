# Appendix F — Example Production Meeting Minutes (Google SRE Book)

Source: https://sre.google/sre-book/production-meeting/ (SRE Book, 2017, CC BY-NC-ND 4.0)

**Scope caveat:** This appendix is only a filled-in sample of meeting minutes (dated 2015-10-23), with no prose explanation. It uses joke content ("Goat Teleporter", "flux capacitor", "New Sonnet"). Principles below are *inferred from the template's structure and content*. The reasoning behind production meetings is in Chapter 31, "Communication and Collaboration in SRE", which this appendix illustrates.

## Section headings (the template, in order)

1. **Date** / **Attendees**
2. **Announcements**
3. **Previous Action Item Review**
4. **Outage Review**
5. **Paging Events**
6. **Nonpaging Events**
7. **Monitoring Changes and/or Silences**
8. **Planned Production Changes**
9. **Resources**
10. **Key Service Metrics**
11. **Discussion / Project Updates**
12. **New Action Items**

## Core principles (inferred)

- **Review past action items before new business.** "Previous Action Item Review" comes right after announcements, so commitments are checked every week.
- **Outages are tied to the error budget.** The outage entry ends with its budget consequence: a production freeze.
- **Unactionable pages are a problem to fix in the meeting.** A noisy alert gets a concrete threshold change with an owner and a bug.
- **Pages and non-pages are separate agenda items.** Both are reviewed, even when a section says "None".
- **Monitoring changes and silences are recorded openly.** They have their own section, so changes to alerting can be seen and audited.
- **SLOs are reported against target over a fixed window.** Each metric shows its status, its value, the target and the window.
- **Every follow-up has an owner.** Action items use `TODO(username)`.

## Concrete numbers / thresholds (exact, as written)

| Item | Value as written |
|---|---|
| Meeting date | 2015-10-23 |
| Outage | #465 ("New Sonnet"), "blew through error budget" |
| Impact | "1.21B queries lost due to cascading failure" |
| Freeze | "pushes to prod frozen for 1 month" |
| Paging alert | `AnnotationConsistencyTooEventual`: "paged 5 times this week" |
| Threshold change | "acceptable delay threshold raised from 60s to 180s" (bug 4821600) |
| Maintenance | USA-1 offline "between 2015-10-29 and 2015-11-02" |
| Utilization now | "60% of CPU, 75% RAM, 44% disk" |
| Utilization last week | "40%, 70%, 40%" |
| Latency SLI | "OK 99ile latency: 88 ms < 100 ms SLO target [trailing 30 days]" |
| Availability SLI | "BAD availability: 86.95% < 99.99% SLO target [trailing 30 days]" |
| Upcoming launch | Project Molière "launching in two weeks" |

Notes on the numbers:
- The SLO window is a **trailing 30 days**, not a calendar month.
- The latency SLO is on the **99th percentile** ("99ile"), not the mean.
- The availability SLO in the example is **99.99%** (four nines). The measured 86.95% marks it **BAD**.
- The freeze is **1 month**. An exception is possible only "on grounds that event was bizarre & unforeseeable", and the consensus is that one is unlikely.

## Recommended practices & procedures (as shown)

- **Outage write-up content:** impact (queries lost), the combined root causes, the fix that was deployed (with a bug ID), and the prevention work (load balancing and load shedding, each with its own bug ID). The root causes were a latent leaked-file-descriptor bug on searches with no results, a missing corpus entry, and unprecedented traffic.
- **Error budget enforcement:** an exhausted budget triggers a release freeze. Exceptions go through a named person (docbrown) and need a stated justification.
- **Noisy paging alert procedure:** note how often it paged and the likely cause (cross-regional Bigtable replication delay). Keep the investigation going under a bug. If no fix is expected soon, raise the acceptable threshold "to reduce unactionable alerts". Record the change under Monitoring Changes and add an owned TODO.
- **Planned changes:** list maintenance windows and say whether any response is needed. Here, "No response required, traffic will automatically route to other clusters in region."
- **Resources:** track resources borrowed for an incident and commit to returning them ("will spin down additional server instances and return resources next week"). Report utilization next to last week's numbers to show the trend.
- **Action items:** one owner each, matching the commitments made in earlier sections:
  - `TODO(martym): Raise AnnotationConsistencyTooEventual threshold.`
  - `TODO(docbrown): Return instance count to normal and return resources.`

## Anti-patterns implied

- Leaving alerts that page but cannot be acted on. The example fixes this explicitly.
- Shipping after the error budget is gone. Pushes are frozen instead.
- Treating an outage as fully closed once the immediate bug is fixed. Prevention work is tracked separately.
- Keeping borrowed incident capacity indefinitely.
- Changing monitoring or adding silences without recording it.

## Template / checklist (reusable)

```
Date:            Attendees:
Announcements:
Previous Action Item Review:   <item (bug)> — status
Outage Review:   <name (outage #)> — impact; root cause(s); fix (bug); prevention (bugs); error-budget consequence
Paging Events:   <alert>: paged N times; likely cause; bug; plan
Nonpaging Events:
Monitoring Changes and/or Silences:   <alert>, <old> -> <new>, bug, TODO(owner)
Planned Production Changes:   <what/when>; response required?
Resources:       borrowed/returned; utilization CPU/RAM/disk (vs last week)
Key Service Metrics:   OK|BAD <SLI>: <value> vs <target> SLO [trailing 30 days]
Discussion / Project Updates:
New Action Items:   TODO(owner): ...
```

## Short verbatim quotes

- "Major outage (#465), blew through error budget"
- "Annihilated availability error budget; pushes to prod frozen for 1 month"
- "will raise acceptable consistency threshold to reduce unactionable alerts"
- "OK 99ile latency: 88 ms < 100 ms SLO target [trailing 30 days]"
- "No response required, traffic will automatically route to other clusters in region."
