# Data Integrity, Distributed State, Scheduling and Pipelines

Sources: SRE Book ch.23 "Managing Critical State: Distributed Consensus for Reliability", ch.24 "Distributed Periodic Scheduling with Cron", ch.25 "Data Processing Pipelines", ch.26 "Data Integrity: What You Read Is What You Wrote". SRE Workbook ch.13 "Data Processing Pipelines".

## Data integrity (Book ch.26)

- **Users care about restores, not backups.** Set SLOs for data availability and recovery time, and prove you can meet them.
- **A backup can be loaded back into the application; an archive can't.** Archives exist for compliance.
- **The failure space has 24 combinations**: 6 root causes (user action, operator error, application bug, infrastructure defect, hardware failure, site disaster) × 2 scopes (wide or narrow) × 2 rates (big-bang or creeping).
- **Defense in depth:**
  1. **Soft deletion** (lazy delete), with a delay that is typically 15, 30, 45 or 60 days. Most problems are found within 60 days, so longer delays add little.
  2. **Backups and recovery**, tiered by restore speed. Near the live store, restores take minutes and copies are kept for hours to days. On the site's distributed filesystem, restores take hours and copies are kept for days. Offline (tape or offsite) is slowest. Google keeps **30–90 days** of backups.
  3. **Early detection**: out-of-band validation pipelines. Gmail's validators run daily and catch problems within 24h.
- **Replication is not a backup.** It copies corruption and deletions too. Recovery plans must not depend on it.
- **24 hours is a reasonable starting threshold for "too long"** to notice a data problem. Even 99.99% good bytes in 2 GB means up to 200 KB of garbage.
- **Alert on deletion rates globally** (for example about 10× the observed p95), not per user.
- **Test restores end to end, automatically, and often.** "You only know that you can recover your recent state if you actually do so."

## Distributed consensus and critical state (Book ch.23)

- Leader election, group membership, distributed locks, leases, reliable queues and shared configuration are all **consensus problems**. Use a proven Paxos or Raft implementation such as etcd, ZooKeeper or Chubby-like services.
- **Heartbeats, STONITH, gossip and human-driven failover are "conceptually unsound"** substitutes. They cause split-brain.
- **Replica counts**: 2f+1 replicas tolerate f failures, and Byzantine faults need 3f+1. The minimum is 3, but **5 is recommended**, because planned maintenance causes most downtime. Five replicas survive losing 40%; six survive only 33%, so an even count is worse. Add capacity once you're down to 3 of 5.
- **Performance**: each consensus operation needs a round trip to a quorum. That is about 1 ms within a datacenter, about 45 ms across the US and about 70 ms from NY to London. A 10 ms disk write caps you at about 100 ops/s per leader. Batch, pipeline, and place replicas close to clients.
- **Use renewable leases with timeouts**, not locks held indefinitely. Use randomized backoff so proposers don't keep colliding. FLP means guaranteed progress in bounded time is impossible, but safety is still guaranteed.
- **Monitor**: whether a leader exists, how often it changes (a falling term or view number suggests a bug), transaction-number progress, lagging replicas, and quorum health.

## Distributed cron (Book ch.24)

- **Fail closed by default**: skip a launch rather than risk running it twice. A missed run is usually easier to fix than a duplicate, for example a newsletter sent twice. Job owners monitor their own jobs.
- **Launch protocol**: the leader writes "launch started" to a Paxos quorum and waits for confirmation, then launches. It writes "launch finished" the same way, whether the launch succeeded or failed.
- **Launch identity = job + scheduled time.** Use deterministic job names so a new leader can find in-flight launches. Recovering a half-finished launch requires the external operation to be idempotent, or its state to be queryable.
- Leader election should finish in under a minute, the smallest cron interval. A leader that loses leadership must stop acting immediately.
- **Avoid thundering herds**: let the system pick start minutes (crontab `?`) by hashing the job.

## Data pipelines (Book ch.25, Workbook ch.13)

- **SLOs are end-to-end, and efficiency is measured per stage.** Customers care about the whole pipeline, and per-stage SLOs add alert noise.
- **Pipeline SLI types**: freshness, correctness, coverage/completeness and throughput.
  - Freshness can be stated as X% of data processed within Y, oldest unprocessed data under Y, or job completed within Y.
  - Examples: 99% of score-changing actions shown within 30 min; at most 0.1% of invoices wrong per quarter, checked against test accounts with known expected results.
- **"Stale data is almost always better than incorrect data."** Gate downstream stages until their input is complete.
- **Periodic pipelines**: an interval must be well above the run time. A 20-minute job shouldn't run "much below" every 40 minutes. If the work is continuous, or will grow to be, use a continuous pipeline rather than a periodic one.
- **Don't kill and restart a hung chunk** if there's no checkpointing, because everything restarts. Don't add workers to a late job either, since that adds load and can make the thundering herd worse.
- **Watch shared resources.** Moiré load patterns appear when periodic pipelines collide.
- **Idempotent work units** remove the need for "exactly once".
- **Rollout**: canary the whole pipeline, or stage the output and verify it before committing (two-phase mutation). Ramp from a 1% dry run through 1%, 10%, 50% to 100%.
- **Workflow-style correctness guarantees** (Book ch.25): configuration tasks act as barriers, a worker must hold a valid lease to commit, output files have unique names, and the server token is checked on every operation.

## Checklist

- [ ] Data recovery SLO defined; restore tests automated and alerting on failure
- [ ] Soft delete, then tiered backups (30–90 days), then validation pipelines; replication not counted as a backup
- [ ] Deletion-rate and corruption detection within about 24h
- [ ] Leader and lock state on a consensus system with 5 replicas (3 at minimum) across failure domains; leases rather than permanent locks
- [ ] Monitoring of leader existence and churn, and replica lag
- [ ] Cron: fail closed, idempotent or queryable launches, jittered start times
- [ ] Pipelines: end-to-end freshness and correctness SLOs, gated stages, canaried rollout
