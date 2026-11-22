# Appendix C — Example Incident State Document (Google SRE Book)

Source: https://sre.google/sre-book/incident-document/
Example document title: **"Shakespeare Sonnet++ Overload: 2015-10-21"**

**Scope note:** This appendix is only a filled-in template. It has no explanatory prose of its own: no rationale, no anti-patterns section, no discussion. Everything below comes from the document's own fields and inline instructions. Points marked *[inferred]* are the reader's interpretation of how the template is built, not text from the appendix. It goes with Chapter 14, "Managing Incidents" (which uses the same Shakespeare scenario), and Appendix D, "Example Postmortem".

## Core principles (as the template embodies them)

- **One living document holds the current state of the incident.** At the top it links to the process reference ("Incident management info: https://incident-management-cheat-sheet").
- **Someone owns the document.** "(Communications lead to keep summary updated.)" The Communications lead keeps it current.
- **Roles are explicit and named.** The "Command Hierarchy (all responders)" section lists the Current Incident Commander, the Operations lead, the Planning lead and the Communications lead, plus a slot for the **Next Incident Commander**.
- **One person can hold several roles.** In the example, `jennifer` is Incident Commander, Planning lead and Communications lead at once, and `docbrown` is Operations lead. *[inferred]* Roles are separate jobs, but on a small incident they do not each need their own person.
- **Handoff is planned ahead.** "Next Incident Commander: to be determined" is a standing field.
- **"Done" is defined explicitly through exit criteria** measured against SLOs, not left to gut feel.

## Concrete numbers / thresholds (exact, as written)

| Item | Value in document |
|---|---|
| Incident number | **#465**, Status: **active** |
| Command post | **#shakespeare on IRC** |
| Detailed-status update cadence | "Update **at least every four hours** and at handoff of Comms Lead role." |
| Availability SLO (exit criterion) | **99.99%** |
| Latency SLO (exit criterion) | **99%ile < 100 ms** |
| Required stable duration | Within SLOs "for **30+ minutes**" |
| Capacity action | "Increasing serving capacity globally by **2x**" |
| Tracked bug | **Bug 5554823** (flux capacitor load balancing) |
| Timezone | All times in **UTC**; timeline is **most recent first** |
| Detailed status last updated | 2015-10-21 15:28 UTC by jennifer |

Correct reading of the update cadence: the four-hour minimum applies to the **Detailed Status** section, and that section must also be updated **at every Comms Lead handoff**. The appendix does not give a four-hour rule for the summary or the timeline.

Timeline (as recorded, shown oldest first here for readability; the document itself lists newest first):
- 14:55 docbrown: pager storm, `ManyHttp500s` in all clusters
- 15:01 docbrown: incident declared due to cascading failure (6 minutes after the first pages)
- 15:04 martym: gets the text of the newly discovered sonnet from the shakespeare-discuss@ mailing list
- 15:10 martym: adds the new sonnet to the corpus and starts the index MapReduce
- 15:21 jennifer: sends all traffic to the **USA-2 "sacrificial cluster"** and drains the other clusters "so they can recover from cascading failure while spinning up more tasks"; MapReduce index job complete, waiting for Bigtable replication to all clusters
- 15:28 jennifer: increases serving capacity globally by 2x

## Recommended practices & procedures (from the template's structure)

1. **Header block:** a short Summary ("Shakespeare search service in cascading failure due to newly discovered sonnet not in search index"), Status with incident number, and Command Post(s).
2. **Command Hierarchy:** list every responder role with a named owner, and keep a Next IC field so handoff is always being planned.
3. **Detailed Status, timestamped with the author's name:** record who updated it last and when.
4. **Exit Criteria, each marked TODO/DONE:** one fixes the root problem ("New sonnet added to Shakespeare search corpus"), and one checks the SLOs ("Within availability (99.99%) and latency (99%ile < 100 ms) SLOs for 30+ minutes").
5. **TODO list and bugs filed, each marked DONE/TODO:** link longer-term work to a bug number. In the example, "Run MapReduce job to reindex Shakespeare corpus" (DONE), "Borrow emergency resources to bring up extra capacity" (DONE), "Enable flux capacitor to balance load between clusters (Bug 5554823)" (TODO).
6. **Incident timeline:** most recent entry first, times in UTC, each entry with a timestamp and the responder's name. *[inferred]* This also serves as raw material for the postmortem in Appendix D.
7. **Mitigation pattern shown:** concentrate traffic on a sacrificial cluster so the other clusters can drain and recover, then add capacity (borrowed emergency resources, a global 2x increase).

## Anti-patterns

The appendix names **no** anti-patterns. *[inferred]* By contrast, the template guards against roles nobody owns, a status that has gone stale (hence the update cadence), an unclear end to the incident (hence SLO-based exit criteria), and unclear handoffs (hence the Next IC field and updates at Comms handoff). Chapter 14 covers the explicit anti-patterns, such as freelancing and poor communication.

## Template / checklist (reusable skeleton)

```
<Incident title>: <YYYY-MM-DD>
Incident management info: <link to process cheat sheet>
(Communications lead to keep summary updated.)
Summary: <one line>
Status: <active|...>, incident #<n>
Command Post(s): <channel>
Command Hierarchy (all responders)
  Current Incident Commander: <name>
    Operations lead: <name>
    Planning lead: <name>
    Communications lead: <name>
  Next Incident Commander: <name | to be determined>
(Update at least every four hours and at handoff of Comms Lead role.)
Detailed Status (last updated at <UTC time> by <name>)
  Exit Criteria:
    - <root-cause fix>                      TODO/DONE
    - Within <availability> and <latency> SLOs for <duration>  TODO/DONE
  TODO list and bugs filed:
    - <action> (Bug <id>)                    TODO/DONE
  Incident timeline (most recent first: times are in UTC)
    <YYYY-MM-DD HH:MM UTC> <name>
      <what happened / what was done>
```

## Short verbatim quotes

- "(Communications lead to keep summary updated.)"
- "(Update at least every four hours and at handoff of Comms Lead role.)"
- "Within availability (99.99%) and latency (99%ile < 100 ms) SLOs for 30+ minutes"
- "Incident timeline (most recent first: times are in UTC)"
- "Directing all traffic to USA-2 sacrificial cluster and draining traffic from other clusters"
