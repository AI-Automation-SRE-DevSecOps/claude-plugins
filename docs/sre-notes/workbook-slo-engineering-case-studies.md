# SRE Workbook, Chapter 3: SLO Engineering Case Studies

Source: https://sre.google/workbook/slo-engineering-case-studies/
Two customer stories (Evernote, The Home Depot) from work with Google's CRE team. They are not Google-internal rules; they show non-Google companies fitting SLOs to their own setups.

## Core principles

- **Start from what you promise the customer** (Evernote, "Introduction of SLOs"). Evernote's first SLO covered "the availability of the Evernote service for users to access and sync their content across multiple clients."
- **"Perfect is the enemy of good."** Evernote picked an initial SLO "that covered most, but not all, user interactions" as a proxy for quality of service, and revised it over time.
- **SLOs give ops and dev a common frame of reference.** They take subjectivity out of decisions, so both teams reach similar conclusions from the same facts (Evernote, "Why Did Evernote Adopt the SRE Model?").
- **Customer and cloud provider share responsibility** (Evernote, "Breaking Down the SLO Wall").
- **The business owner (product manager) sets the SLO**, based on business criticality (THD, "Future Aspirations").

## Concrete numbers, thresholds and definitions

**Evernote:**
- SLO: **99.95% uptime measured over a monthly window**, applied to certain services and methods. They deliberately chose a **calendar month rather than a rolling window** to keep service reviews focused and organized.
- What is measured: a built-in status page that exercises most of the stack and returns **HTTP 200** when all is well.
- Prober: a third-party service (Pingdom) located completely outside Evernote's environment, so the load-balancing stack is tested too. It **polls the frontend nodes every minute** from multiple probes in North America and Europe.
- Definition of "down": a failed check marks the node **"Unconfirmed Down"**. A **second, geographically separate prober** then checks. If that also fails, the node counts as down for the SLO, and it stays down while consecutive probes keep failing.
- **Maintenance windows count as downtime**, because many users won't know about published windows.
- Cadence: a **monthly** joint Evernote/Google review of SLO performance, plus a **six-month SLO review cycle**. They had revised the SLOs twice and were on **version 3 after about nine months**.
- GCP promises **99.95% availability for Compute Engine**. A global rollup can stay green while outages in Evernote's region get "lost." Google CRE now tells Evernote things like "causing a 5% impact to Evernote's SLO."

**The Home Depot (THD):**
- **Latency:** use percentiles, not arithmetic averages. Every service must at minimum hit a **90th percentile** target. User-facing services have a preferred target at the **95th and/or 99th percentile**. White-box monitoring must be supplemented with black-box monitoring.
- **Errors:** never signal an error in the body of a 2xx response. Service faults (e.g., out of memory) return **5xx**; client faults (e.g., a malformed request) return **4xx**. THD tracks both 4xx and 5xx, but **only 5xx count toward SLOs**.
- **Volume:** track average RPS, peak RPS and total volume over the reporting period, and let each service pick one. The SLO is set to **expected peak capacity** (Black Friday).
- **No utilization SLO.** Utilization is still monitored at 1-minute granularity and still used for capacity planning.
- Scale: about **50 services** at the start of the year, **800** by the end, with about **50 new services registered per month**. That is "0 to 800 SLO-supported services in less than a year."
- Data pipeline: TPS Reports produces **hourly** VALET metrics from BigQuery. The VALET service trends SLOs at **daily, weekly and monthly** granularity.
- **Simplified uptime tiers** for product managers:
  - 99.5%: apps not used by store associates, or an MVP of a new service
  - 99.9%: enough for most nonselling systems
  - 99.95%: selling systems, or services that support them
  - 99.99%: shared infrastructure services

## The VALET template (THD)

| Letter | Service question | Batch adaptation |
|---|---|---|
| **V**olume (traffic) | How much business volume can my service handle? | Volume of records processed |
| **A**vailability | Is the service up when I need it? | How often (%) the job completed by a certain time |
| **L**atency | Does the service respond fast when I use it? | Time it takes for the job to run |
| **E**rrors | Does the service throw an error when I use it? | Records that failed to process |
| **T**ickets | Does the service require manual intervention to complete my request? | Times an operator manually fixes data and reprocesses a job |

VALET extends the Four Golden Signals. It drops utilization and adds availability (tracked separately from errors) and tickets, which THD kept for historical reasons.

## Recommended practices and procedures

**Evernote's first SLO document had four parts:** (1) the SLO definition; (2) what to measure; (3) how to measure it; (4) how to calculate the SLO from the monitoring data, including edge cases such as maintenance windows.

**Using the SLO:**
- Use the error budget to allocate resources. A missed SLO raises the priority of related fixes.
- In the monthly review, deep-dive on outages and set action items that the root-cause process missed.
- Share your SLO and live performance against it with your cloud provider, so both sides watch the same dashboards. Agree that SLO impact above some level gets P1 treatment with special handling and a shared bridge.
- Balance what you would *like* to measure against what is *possible* to measure.

**THD's four-part adoption strategy:**
1. **Common vernacular:** define SLOs in THD's own context and measure them the same way everywhere.
2. **Evangelism:** start with senior leadership, then meet teams one by one. Send a weekly VALET report to leadership. Use early adopters, tech talks, workshops and FiRE Academy training.
3. **Automation:** collect SLIs automatically for every service deployed to production.
4. **Incentive:** give development managers annual goals to set and measure SLOs, which fed into their performance reviews.

**Further practices:**
- Each microservice publishes availability and latency SLOs for the API calls other microservices make to it (e.g., Cart depends on Inventory).
- Hold regular SLO reviews, weekly or monthly. The outcome is either action items to get back within the SLO or a change to an unrealistic SLO.
- Planned, not yet done: an error-budget feature freeze when a service is out of SLO, and deployment gating on VALET.

## Anti-patterns warned against

- Waiting for a "perfect" SLO that covers every interaction. It "would take many months to achieve."
- Basing your SLOs only on the provider's global SLO and metrics. Regional impact disappears in the rollup.
- Leaving maintenance windows out of the SLO when users don't know about them.
- Using support tickets as the only reliability measure (THD's starting point), with scattered dashboards that keep no history.
- Returning errors in a 2xx response body, or using averages instead of percentiles for latency.
- Probing in a way that lets random internet issues cause false positives (hence the second-prober confirmation).
- A known limitation of THD's design: alert thresholds in its monitoring systems "aren't integrated with SLOs."
- Changing SLOs too often, or letting them go stale.
- Thinking SLOs require THD-scale automation: "there are benefits to just writing SLOs in the first place."

## Short verbatim quotes

- "Perfect is the enemy of good." (Evernote)
- "our SLO calculations treated maintenance as downtime." (Evernote)
- "an error budget/SLO approach has led both teams to make similar decisions when presented with the same facts" (Evernote)
- "SLOs are best leveraged as a trending tool" (THD)
- "we went from 0 to 800 SLO-supported services in less than a year." (THD)
- "SLO culture is an ongoing process and not a one-time fix or solution." (Conclusion)
