# SRE Book, Chapter 16: Tracking Outages

Source: https://sre.google/sre-book/tracking-outages/ (written by Gabe Krabbe, edited by Lisa Carey). Sections: intro, Escalator, Outalator (Aggregation, Tagging, Analysis, Reporting and communication, Unexpected Benefits), sidebar "Building Your Own Outalator".

## Core principles

- **You need a baseline before you can improve reliability.** Reliability only improves over time if you "start from a known baseline and can track progress" (intro).
- **Postmortems don't cover everything.** They are "only written for incidents with a large impact", so they miss issues that are individually small but frequent and widespread. They also tend to stay inside one service, which hides fixes with a "poor cost/benefit ratio" for one case "but that would have large horizontal impact" (intro; footnote 84 gives a Bigtable example: a costly change with a small benefit for one outage pays off if it helps across many events).
- **Ingest passively, change nothing for users.** Escalator and Outalator were both added on top of existing infrastructure. Outalator "passively receives all alerts sent by our monitoring systems." Escalator began as a "largely transparent tool" that received copies of emails sent to on-call aliases, so nobody had to change how they worked.
- **Alerts and incidents are different units.** Several alerts can be grouped into one incident, which allows "separate analysis of 'incidents per day' versus 'alerts per day'" (Outalator; Aggregation).
- **Duplicate alerts can't be avoided entirely.** You should try to keep down the number of alerts one event triggers, but "triggering multiple alerts is unavoidable in most trade-off calculations between false positives and false negatives" (Aggregation).
- **Analysis is the main point.** Historical data helps during response ("what did we do last time?"), but it is "far more useful" for systemic, periodic or wider problems. Supporting that analysis is "one of the most important functions of an outage tracking tool" (Analysis).

## Concrete definitions, numbers and mechanics

The chapter contains no SLO math, thresholds or formulas. These are its exact specifics:

- **Escalator:** a central, replicated system shared by all SRE alert notifications. It tracks whether a human has acknowledged receipt. If nobody acknowledges within "a configured interval", it escalates to the next configured destination(s), e.g. primary on-call to secondary. No interval value is given.
- **Terminology:** an escalating notification is an **"alert"**. A group of alerts is an **"incident"**. Alerts included in an email handoff are called "outalations".
- **Tag syntax:** tags are mostly free-form single "words". **Colons are semantic separators**, which encourages hierarchical namespaces. The main suggested prefixes are **"cause"** and **"action"**, and each team's suggestion list is generated from its own historical usage. Examples: `cause:network`, `cause:network:switch`, `cause:network:cable`, `customer:132456`, `bug:76543` (parsed into a link to the bug tracker), `bogus` (widely used for false positives).
- **Analysis layers** (Analysis):
  1. Bottom layer: counting and basic aggregate statistics, e.g. incidents per week/month/quarter and alerts per incident.
  2. Next layer ("more important, and easy to provide"): comparisons between teams/services and over time to find first patterns and trends, so you can tell whether a load is "normal". Example: "That's the third time this week" only means something once you know whether it used to happen five times per day or five times per month.
  3. Next step: wider issues that need semantic analysis, e.g. which infrastructure component causes the most incidents.
- **Example questions to answer:** "How many alerts per on-call shift does this team get?", "What's the ratio of actionable/nonactionable alerts over the last quarter?", "Which of the services this team manages creates the most toil?"
- **Production review cadence:** periodic reviews happen "weekly for most teams".

## Recommended practices and procedures

- **Merge queues into one view.** Show a time-interleaved list of notifications across several queues. This helps because one SRE team is often the primary contact for services that have different secondary escalation targets, usually dev teams.
- **Keep the full record.** Store the original notification and silently capture email replies. Let people mark annotations as **"important"** so the rest of the thread collapses. This gives more context than a fragmented email thread.
- **Group alerts into incidents in the tool, not by email.** An "it's the same thing" email works for a single alert, but it doesn't scale within a team, across teams, or over time.
- **Tag at any level**, using team-evolved, namespaced tags (`cause:*`, `action:*`).
- **Shift handoff:** pick zero or more outalations and email their subjects, tags and important annotations to the next on-call engineer, plus an arbitrary cc list.
- **Report mode** for weekly production reviews: important annotations are expanded inline to give a "quick overview of lowlights".
- **Look across teams for systemic causes.** For example, "stale data" and "high latency" alerts on different teams may share one cause (network congestion causing database replication delays). Sometimes the conditions are within the SLO but fall short of what users expect. In that case the fix "may be the introduction of more artificial failures to stop over-performing."
- **Check cross-team status.** If your service looks hit by a Bigtable incident but the Bigtable SRE team hasn't been alerted, page them manually.
- **Use it as a system of record.** Create dummy escalator configurations that page no human, so events still show up in the tool and can be tagged and audited. Examples: use of privileged or role accounts (technical audits, not legal ones), and runs of periodic jobs that may not be idempotent, such as automatic schema changes applied from version control.
- **Build your own:** hook into Slack/Hipchat/IRC.

## Anti-patterns and caveats

- **Relying only on postmortems** misses frequent small-impact issues and cross-service opportunities.
- **Handling duplicate alerts one email at a time** doesn't scale.
- **A predetermined tag list.** Some tags will be typos (`cause:netwrok`) or unhelpful (`problem-went-away`), but letting teams find their own standards "will result in a more useful tool and better data."
- **Over-reading "most incidents caused"** (footnote 85). It is a good starting point, but it may be an artifact of over-sensitive monitoring or of a few clients misbehaving or running outside the agreed service level. Incident counts alone say nothing about how hard something is to fix or how severe the impact is.
- **Treating dummy-queue audit logs as legal audits.** The feature is basic and meant for technical audits.

## Checklist (derived from the chapter)

- [ ] Every alert is captured passively, with acknowledgment tracking and escalation from primary to secondary
- [ ] Unified multi-queue view; replies stored; "important" annotations
- [ ] Alerts can be grouped into incidents, with metrics for both alerts/day and incidents/day
- [ ] Namespaced free-form tags (`cause:`, `action:`, `bug:` links, `bogus`)
- [ ] Reports: incidents per week/month/quarter, alerts per incident, alerts per shift, actionable/nonactionable ratio, toil by service
- [ ] Trends compared across teams and over time
- [ ] Handoff email at shift change; report mode for weekly production review
- [ ] Dummy queues for audit/system-of-record events

## Verbatim quotes

- "Improving reliability over time is only possible if you start from a known baseline and can track progress."
- "As trivial as tagging appears, it is probably one of the Outalator's most useful unique features."
- "Improved cross-team visibility can and does make a big difference in incident resolution, or at least in incident mitigation."
- "the number of incidents alone gives no indication as to the difficulty to fix or severity of impact."
