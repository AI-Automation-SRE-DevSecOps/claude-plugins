# SRE Book, Chapter 10: Practical Alerting from Time-Series Data

Source: https://sre.google/sre-book/practical-alerting/ (by Jamie Wilkinson, edited by Kavita Guliani). Most of the chapter describes **Borgmon**, Google's internal time-series monitoring system, built shortly after Borg (2003). The chapter names Prometheus as the open-source tool closest to it, especially in its rule language. Riemann, Heka and Bosun are also listed.

## Core principles

- **Alert on aggregates, not on single machines** (intro): "being alerted for single-machine failures is unacceptable because such data is too noisy to be actionable." Systems should "aggregate signals and prune outliers" while keeping the detail needed to inspect individual components.
- **Look at distributions, not averages** (intro): understand the spread of response times across servers so you can find what drives the latency tail.
- **Treat time-series collection as the core job** (intro, Rise of Borgmon): instead of per-target check scripts, use a common exposition format, collect everything at scale, and evaluate rules centrally. The same data feeds charts and alerts, and stored history can be used in alert computation. This is **white-box monitoring**.
- **Use collection failure as a signal** (Collection of Exported Data): synthetic per-target variables let rules detect unavailable tasks.
- **Pair white-box with black-box monitoring** (Black-Box Monitoring): white-box alone means "you aren't aware of what the users see"; "You can only alert on the failures that you expected."
- **Maintenance cost should grow sublinearly** (Ten Years On): "Ensuring that the cost of maintenance scales sublinearly with the size of the service is key." Separating rules from targets lets monitored-system size scale independently of the alerting rules.

## Concrete numbers, thresholds and formulas (as written)

- **Arena horizon:** datacenter and global Borgmon are "typically" sized for **about 12 hours** of data, with much less in the lowest-level collector shards. Footnote 50 calls 12 hours "a magic number" chosen to keep enough data in RAM for debugging an incident.
- **Memory:** about **24 bytes per data point**, which means **1 million unique time-series for 12 hours at 1-minute intervals fits in under 17 GB of RAM**.
- **Topology:** typically **one Borgmon per cluster and a pair at the global level** (Collection section), or "**two or more global Borgmon** ... and one Borgmon in each datacenter" (Sharding section). Very large services add scraper-only shards below the datacenter level.
- **Minimum labels** for a time-series to be identifiable: `var`, `job`, `service`, `zone`. The example expression also has `instance`: `{var=http_requests,job=webserver,instance=host0:80,service=web,zone=us-west}`.
- **`rate()`** = total delta ÷ total time between the earliest and latest values in the window. The `[10m]` range protects against missed collections.
- **Sum of rates, not rate of sums** (footnote 52): this guards against counter resets and missing data.
- **Pending duration:** "Typically, this duration is set to **at least two rule evaluation cycles**" so that missed collections don't cause false alerts.
- **Example page:** error ratio over 10 minutes **> 0.01 (1%)** AND error rate **> 1 per second**, `for 2m`, `severity=page`. In the worked data the ratio is 0.15, but the alert stays inactive until errors exceed 1/s. It then goes **pending** for two minutes before it **fires**.
- **Worked example values:** per-task request rates of 1, 0.9, 1.1, 0 and 1 sum to a datacenter rate of 4. Errors of 0.6 against 4 requests give a ratio of 0.15.

## Recommended practices and procedures

- **Prefer counters to gauges** (Rule Evaluation, Tip). A counter is "monotonically non-decreasing" and keeps its meaning between samples. A gauge can miss activity that happens between samples.
- **Use the naming convention `level:variable:operation`** (for example `task:http_requests:rate10m`, `dc:http_errors:ratio_rate10m`): aggregation level, variable name, and the operation that produced it.
- **Aggregate in steps**: per-task rate → `sum without instance` → filter out non-200 codes (`code=!/200/`) → divide errors by requests → compare with the SLO.
- **Spread scrapes across the interval** so targets are not collected in lockstep. Use service discovery to build target lists.
- **Route by severity** (Alerting): page-worthy alerts go to on-call, important but subcritical alerts go to ticket queues, and everything else stays as informational dashboard data.
- **Alertmanager functions**: inhibit some alerts while others are active, deduplicate the same labelset coming from multiple Borgmon, and fan in or fan out alerts with similar labelsets.
- **Shard the hierarchy**: use a streaming protocol between tiers. Upper tiers filter what they import so per-task series don't fill the global arena. Two or more global replicas give diversity across maintenance zones.
- **Probe from two vantage points** (Black-Box Monitoring): probe both the load-balanced frontend (e.g. www.google.com) and the per-datacenter backends behind it. This isolates localized failures and lets you suppress alerts. Probers can validate response payloads and export latency histograms.
- **Manage rules as code** (Maintaining the Configuration): keep rules separate from targets, use templates and macros, write unit and regression tests on synthesized time-series, run CI, and have each Borgmon validate config before accepting it.
- **Group aggregations by shared fate**: tasks share configuration, jobs in a shard share a datacenter, and sites share networking.

## Anti-patterns Google warns against

- Paging on single-machine failures.
- Collecting all tasks globally into one place, which creates a scaling bottleneck and a single point of failure.
- Relying only on white-box monitoring: DNS-failed queries are invisible, and queries lost in a server crash "never make a sound."
- Using gauges where counters would work, which misses events between samples.
- Taking the rate of sums instead of the sum of rates.
- Alerting with no minimum duration, which lets alerts "flap."
- Untested or duplicated rule configuration. The schemaless varz interface "requires careful change management."

## Templates and checklists

**Alert rule template:**
```
{var=dc:http_errors:ratio_rate10m,job=webserver} > 0.01
  and by job, error
{var=dc:http_errors:rate10m,job=webserver} > 1
  for 2m
  => ErrorRatioTooHigh
    details "webserver error ratio at %trigger_value%"
    labels { severity=page };
```

**Synthetic per-target variables:** was the name resolved to a host and port; did the target respond to collection; did it respond to a health check; what time did collection finish.

**Two classes of config library:** (1) templates that codify the varz schema of a code library (HTTP server, memory allocation, storage client, generic RPC); (2) generic aggregation templates that roll data up from a task to the global service.

**Three uses of labels:** breakdowns of the data itself (e.g. `code`), the source of the data (`instance`, `job`), and locality or aggregation (`zone`, `shard`).

## Short quotes

- "May the queries flow, and the pager stay silent." (Traditional SRE blessing, epigraph)
- "Borgmon is really just a programmable calculator, with some syntactic sugar that enables it to generate alerts."
- "Aggregation is the cornerstone of rule evaluation in a distributed environment."
- "Borgmon allows engineers to write smarter alerting rules by using the collection failure itself as a signal."
