# SRE Workbook, Chapter 13: Data Processing Pipelines

Source: https://sre.google/workbook/data-processing/. Authors: Rita Sodt and Igor Maravić (Spotify), with Gary Luo, Gary O'Connor, and Kate Ward.

## Core principles
- **Run the pipeline as a service, with an SLO.** "Pipeline Best Practices" covers pipelines responsible for "correctly processing data in a timely manner for consumption by other systems".
- **Measure SLOs end-to-end, efficiency per stage.** Customers only care about the SLO for the sum of all stages ("End-to-end measurement"). Efficiency and resource usage, by contrast, "should be measured at each individual stage" ("Implement Autoscaling and Resource Planning").
- **Stale beats wrong.** "Stale data is almost always better than incorrect data" ("Delayed data"). A pipeline should stall and wait for its inputs, not process incomplete data.
- **Plan for dependency failure.** At a minimum, design for "the largest failure accounted for in their advertised SLAs" ("Plan for Dependency Failure"). Sometimes this means offering a looser SLA to your own customers.
- **Automate before you page.** Design the pipeline so that a machine or zonal outage "never triggers an SLO violation page". A page should mean all automated measures have been exhausted ("Plan Escalation Paths").

## Concrete numbers, formulas and definitions
- **Freshness SLO formats** ("Data freshness"):
  - "X% of data processed in Y [seconds, days, minutes]."
  - "The oldest data is no older than Y [seconds, days, minutes]."
  - "The pipeline job has completed successfully within Y [seconds, days, minutes]."
  - Example: Shave the Yak targets 99% of score-impacting user actions reflected in the scoreboard within 30 minutes.
- **Correctness SLO** ("Data correctness"):
  - Example: no more than 0.1% of invoices incorrect per quarter.
  - A second option is a target on hours/days that bad data is served.
  - Generate "golden data" (for example from test accounts) and compare expected output with actual.
- **Rollout ramp** ("Testing with a 1% dry run", "Performing a partial deployment"):
  - Start with a 1% dry run of production data in a nonproduction environment.
  - For a partial deployment, start with one or two accounts, then ~1%, ~10%, ~50%, and finally 100%.
- **Maturity matrix** (Table 13-2):
  - Each characteristic is scored 1–5. 1 = "Chaotic" (unplanned, ad hoc, risky, fully manual), 3 = "Functional", 5 = "Continuous improvement".
  - If more than one milestone applies, use the score in between (2 or 4).
  - Five characteristics: failure tolerance; scalability; monitoring and debugging; transparency and ease of implementation; unit and integration testing.
- **Global work scheduling** (Table 13-2):
  - Level 3 is hot/hot/hot: the same work runs in all three regions.
  - Level 5 is warm/warm/warm: work is distributed across all three regions and stored centrally. Note that hot/hot/hot is *not* the top level.
- **Spotify numbers:** "hundreds of billions of events" daily; clients buffer offline up to 30 days; components provisioned for **50% CPU at peak**; hour-13 bucket closed at 14:53 = 53-minute delay.
- **Spotify SLO definitions:**
  - *Timeliness* is the maximum delay in delivering an hourly bucket. The delay is measured from the earliest theoretical close time to the actual delivery time. It has three priority tiers: high, normal and low.
  - *Skewness* is the "maximal percentage of data that can be misplaced on a daily basis".
  - *Completeness* is the "percentage of events that are delivered after they are successfully published".
  - Skewness and completeness have SLOs only for finance-bearing events; everything else is best effort.
  - Both are reported daily, from an audit that compares published and delivered event counts.
  - There is no SLO on data quality or accuracy.
- **Event timestamps:** events are assigned to buckets by server receive time, not client time.

## Recommended practices
- **Data isolation and priority:** separate queues or jobs per priority; workers take the highest-priority task; retry failed work on higher-provisioned workers.
- **Disaster recovery:** practice regularly. Google's DiRT simulates regional outages; pipelines should fail over automatically.
- **Documentation**, three categories: system diagrams with live status links; process docs, generated from source where possible; a playbook entry per alert, linked in the alert.
- **Development lifecycle:** prototype (unit tests) → 1% dry run → staging on representative prod data with A/B diff against known-good output → canary the whole pipeline (skip prod writes or use two-phase mutation) → partial deployment via flag → production with fast rollback and bad-data marking.
- **Canarying multihomed or replicated pipelines** (Dataproc/Dataflow): you can't canary one region in isolation. Roll out to a small percentage of data, or run in dry-run mode, instead.
- **Hotspotting:** block fine-grained records, split large work units, keep an emergency flag to skip matching input; spread load, reduce lock granularity.
- **Cost:** account for storage, network and cross-region replication costs, and prune unused data periodically.
- **Security** ("Adhere to Access Control and Security Policies"):
  - Avoid PII in temporary storage, and encrypt it if you must store it.
  - Give each stage least-privilege access.
  - Put TTLs on logs and PII.
- **Design patterns:** idempotent mutations (same result when reapplied); two-phase mutation (stage, verify, then apply); checkpointing (also helps with preemption); shared libraries (e.g. a generic freshness alert); small pipelines instead of a monolith.
- **"Exactly once" is optional** if your work units are idempotent (Table 13-1).
- **Corrupt data response:** (1) stop more corrupt data entering; (2) restore known-good or reprocess (selectively, or from checkpoints); drain region or roll back as needed.
- **Track MTTD and MTTR** for every outage or SLO violation.

## Anti-patterns
- **Per-stage SLOs.** They force tighter per-component alerting that "don't model the user experience", and they miss cross-stage corruption (one stage adds a field that the next stage silently drops).
- Depending on stricter guarantees than a provider advertises; running downstream jobs on incomplete input; promoting corrupt data to low-latency frontends; provisioning for peak 100% of the time; monolithic pipelines.
- **Logging every request (Spotify).** Rate-limit, or log only interesting requests.
- **Unbounded autoscaling (Spotify).** When CPU stops correlating with work, Autoscaler scales "until it uses all of the resources it can find". Fixes: cap instances, restrict daemon CPU, throttle idle components.
- **Major changes during an incident**, unless new code caused it, in which case roll back.

## Templates and checklists
- **Table 13-1, recommended features:** latency (streaming/batch APIs); correctness (exactly-once, two-phase mutations, windowing, black-box monitoring, job gating); high availability (multihoming, autoscaling); MTTR (releases tied to code, tested backup/restore, region drain, useful alerts, checkpointing); MTTD (SLO monitoring, symptom alerts); canary before production; resource accounting dashboard and growth metric; ease of development and operation.
- **ML staleness questions:** is data stuck before preprocessing? Is the model bad (bug, spam, poor features)? Is a stale model running?
- **Spotify documentation questions:** how is delivery enabled, where is data delivered, how is it partitioned, what are the SLOs, what incident support should customers expect?

## Short quotes
- "Stale data is almost always better than incorrect data."
- "customers only care about the SLO for the sum of all stages."
- "you don't need 'exactly once' semantics if your work units are idempotent"
- "event delivery should behave like a postal service: your mail should be delivered on time, intact, and unopened."
- "Debugging components in production by reading through application logs is challenging, and should be a last resort."
