# SRE Book, Chapter 6: Monitoring Distributed Systems

Source: https://sre.google/sre-book/monitoring-distributed-systems/ (Rob Ewaschuk, ed. Betsy Beyer). Scope: what should page a human, and how to handle issues that aren't serious enough to page.

## Core principles

- **Pages are expensive; signal must be high and noise very low** (*Why Monitor?*). Too many pages make engineers "second-guess, skim, or even ignore" alerts, and that can hide a real page.
- **Never alert because "something seems a bit weird"**. The only exception is security auditing of very narrowly scoped components (*Why Monitor?*).
- **Symptoms vs. causes** (*Symptoms Versus Causes*): monitoring answers *what's broken* (the symptom) and *why* (a possibly intermediate cause). Page on symptoms. Use causes as debugging aids. Only worry about causes that are "very definite, very imminent".
- **Black-box vs. white-box** (*Black-Box Versus White-Box*): Google combines "heavy use of white-box monitoring with modest but critical uses of black-box monitoring." Black-box is symptom-oriented and shows active, not predicted, problems. It is "fairly useless" for imminent problems. White-box catches imminent problems and failures hidden by retries, and is essential for debugging. In a layered system, one team's symptom is another team's cause (slow DB reads are a symptom to the DB SRE and a cause to the frontend SRE).
- **Simplicity**: the critical path from problem onset, through the page, through triage and debugging, must stay simple and understandable by everyone on the team. Alerting rules "should be simple to understand and represent a clear failure" (*Setting Reasonable Expectations*).
- **Long-term thinking** (*Monitoring for the Long Term*): every page today takes time away from improving the system. A controlled, short-term drop in availability can be a good strategic trade.

## Definitions (*Definitions*)

- **Monitoring**: collecting, processing, aggregating and displaying real-time quantitative data about a system.
- **White-box**: metrics exposed by the system's internals (logs, JVM Profiling Interface, an HTTP handler that emits internal stats). **Black-box**: testing externally visible behavior as a user would see it.
- **Alert**: a notification meant for a human, sent to a ticket queue, an email alias or a pager. These are called **tickets**, **email alerts** (footnote: "alert spam") and **pages**.
- **Root cause**: a defect that, if repaired, "instills confidence that this event won't happen again in the same way". An incident can have several root causes, and each should be fixed.
- **Push**: any change to a service's running software or its configuration.

## Concrete numbers, thresholds, formulas

- **Staffing**: a Google SRE team of **10–12 members** typically has **one or sometimes two** members whose main job is monitoring. Every team typically has at least one "monitoring person".
- **Four golden signals**: **latency, traffic, errors, saturation**. "If you can only measure four metrics of your user-facing system, focus on these four."
  - *Latency*: keep latency of successful requests separate from latency of failed ones. Fast HTTP 500s distort the overall numbers, and "a slow error is even worse than a fast error". Track error latency; don't just filter errors out.
  - *Traffic*: a high-level, system-specific demand metric, e.g. HTTP req/s (static vs. dynamic), network I/O or concurrent sessions for streaming, transactions and retrievals per second for key-value storage.
  - *Errors*: failures that are explicit (HTTP 500), implicit (a 200 with the wrong content) or by policy ("If you committed to one-second response times, any request over one second is an error"). A load balancer catches complete failures. Only end-to-end tests catch wrong content.
  - *Saturation*: how "full" the service is, focused on the most constrained resource. Systems degrade before 100% utilization, so a utilization target is essential. **Latency increases are often a leading indicator of saturation**. Measuring **99th percentile response time over a small window (e.g., one minute)** gives an early signal. Saturation also covers predictions such as "your database will fill its hard drive in 4 hours."
- **Tail example** (*Worrying About Your Tail*): a service with **100 ms average latency at 1,000 req/s** may have **1% of requests taking 5 seconds**. Across several backends, "the 99th percentile of one backend can easily become the median response of your frontend." Footnote 23: if 1% of requests are 50x the average, the rest are about twice as fast as the average.
- **Histogram buckets**: count requests per latency bucket instead of storing raw latencies: **0–10 ms, 10–30 ms, 30–100 ms, 100–300 ms…**. The boundaries grow roughly exponentially, **by factors of roughly 3**.
- **Resolution** (*Choosing an Appropriate Resolution*):
  - CPU load averaged over a minute won't show even fairly long spikes that drive tail latency.
  - For a target of **no more than 9 hours aggregate downtime per year (99.9% annual uptime)**, probing for a 200 **more than once or twice a minute** is probably too often.
  - For **99.9% availability**, checking disk fullness **more than once every 1–2 minutes** is probably unnecessary.
  - Cheap high resolution: record CPU each second, increment **5%-granularity buckets** each second, and **aggregate every minute**.
- **Pruning**: collection, aggregation and alerting config exercised **less than once a quarter** (for some SRE teams) should be considered for removal.
- **Bigtable case**: the SLO was based on a synthetic client's **mean** performance, which a slow tail distorted ("the worst 5% of requests were often significantly slower"). The fix: the SLO was temporarily set to **75th percentile** latency, email alerts were disabled, and the team worked on performance.
- **Page load reporting**: page frequency is usually expressed as **incidents per shift**, where an incident may be a few related pages. It is reviewed in **quarterly reports with management**.
- Footnote 25: **N + 0** (zero-redundancy) situations and "nearly full" parts of the service count as *imminent*.

## Recommended practices

- If you measure all four golden signals and page when one is problematic (or, for saturation, *nearly* problematic), the service is "at least decently covered".
- Use distributions and percentiles, not means, for latency, CPU and fullness.
- Choose a measurement granularity for each signal. Sample on the server and aggregate outside it.
- To debug, measure both how fast the web server thinks the DB is and how fast the DB thinks it is. That separates a slow DB from a network problem.
- Keep monitoring a standalone system with loosely coupled integrations (e.g. stable web APIs for summary data). Don't merge it with profiling, crash tracking, load testing, log analysis or traffic inspection.
- Replace email alerts with a dashboard of ongoing subcritical problems, optionally paired with a log (*Conclusion*).
- If an alert becomes frequent, remove its root cause. If that isn't possible, fully automate the response.
- Managers and tech leads must prioritize long-term fixes even after the paging pain goes down.

## Anti-patterns

- "Magic" systems that learn thresholds or detect causality automatically. One exception: simple rules on unexpected changes in end-user request rates.
- Complex dependency hierarchies ("if DB is slow, alert DB; else alert website"). These are only used for very stable parts, e.g. "If a datacenter is drained, then don't alert me on its latency."
- Having someone "stare at a screen to watch for problems."
- Mean-based SLOs and alerts, as in the Bigtable case.
- Alerting per task when each task is a tiny fraction of users, as in Gmail's alerts on Workqueue task "de-scheduling".
- Pages with rote, algorithmic responses: "a red flag". If a team won't automate them, it lacks confidence it can clean up tech debt, and that is "a major problem worth escalating."
- Collected signals that no dashboard or alert uses (candidates for removal). Monitoring so complex it is fragile.
- Heroics: high availability kept up by "sheer force of effort" from a few heroic people leads to burnout.

## Checklist: writing or reviewing an alert (*Tying These Principles Together*)

1. Does the rule detect an **otherwise undetected** condition that is **urgent, actionable, and actively or imminently user-visible**?
2. Will I ever be able to ignore it as benign? When and why, and how can I avoid that?
3. Does it **definitely** mean users are hurt? Filter out cases like drained traffic and test deployments.
4. Can I act on it? Is the action urgent or could it wait until morning? Could it be safely automated? Is it a long-term fix or a workaround?
5. Is someone else already paged for this, making one of the pages unnecessary?

Paging philosophy:
- Every page needs urgency, and I can only react with urgency "a few times a day before I become fatigued."
- Every page must be actionable.
- Every page response should require intelligence. A robotic response means it shouldn't be a page.
- Pages should be about a novel problem or an event not seen before.

If a page meets these four, it doesn't matter whether white-box or black-box monitoring triggered it.

## Short verbatim quotes

- "Effective alerting systems have good signal and very low noise."
- "Rules that generate alerts for humans should be simple to understand and represent a clear failure."
- "If a page merely merits a robotic response, it shouldn't be a page."
- "Pages with rote, algorithmic responses should be a red flag."
- "Email alerts are of very limited value and tend to easily become overrun with noise"
- "It's better to spend much more effort on catching symptoms than causes"
