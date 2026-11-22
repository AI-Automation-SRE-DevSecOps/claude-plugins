# Chapter 26 — Data Integrity: What You Read Is What You Wrote

Source: Google SRE Book, https://sre.google/sre-book/data-integrity/ (Raymond Blum, Rhandeev Singh; ed. Betsy Beyer)

## Core principles

- **Definition is user-centric.** Loss, corruption and extended unavailability are "typically indistinguishable to users", so access to data "should remain in perfect shape."
- **Integrity is the means, availability is the goal** (§ Data Integrity Is the Means). Data that is preserved but inaccessible for too long is the same as no data.
- **The secret to superior data integrity is "proactive detection and rapid repair and recovery."**
- **Backups vs. archives** (§ Backups Versus Archives): backups *can* be loaded back into an application; archives *cannot*. Archives serve audit/discovery/compliance and may take, for example, a week to retrieve. Backups must restore "well within the uptime needs of a service."
- **Build a recovery system, not a backup system.** "Backups are a tax" paid for data availability. Teams don't "practice backups". They define **SLOs for data availability across failure modes** and practice meeting them.
- **Replication and redundancy are not recoverability** (§ Challenges of Maintaining Data Integrity Deep and Wide). Replication propagates corrupt rows and errant deletes to every copy.
- **Diversity is key:** protecting against a failure at layer X requires storing data on diverse components at that layer (for example tape versus disk). This is why Gmail had GTape.
- **Defense in depth**, made of three layers plus an overarching one (§ How Google SRE Faces the Challenges):
  1. Soft deletion (lazy deletion for developer APIs)
  2. Backups and related recovery methods
  3. Early detection (regular data validation)
  - Overarching: replication. It is occasionally useful, but "data recovery plans should not rely upon replication."

## Concrete numbers, thresholds, formulas

| Item | Value as written |
|---|---|
| "Too long" for data unavailability | 2011 Gmail incident: 4 days is "perhaps too long". **24 hours** is "a good starting point" for Google Apps |
| Uptime vs. integrity | 99.99% uptime is about 1 hour of downtime a year. 99.99% good bytes in a **2 GB** artifact means **up to 200 KB garbled**, which is catastrophic |
| Failure-mode space | **24 combinations**: Root cause (6: user action, operator error, application bugs, infrastructure defects, faulty hardware, site catastrophes) × Scope (widespread / narrow) × Rate (big bang / creeping) |
| Recovery study | **19** Google data recovery efforts. Most common: deletion or loss of referential integrity from software bugs. Hardest: low-grade corruption found weeks to months later |
| Soft-deletion delay | common choices are **15, 30, 45, or 60 days**. Most hijacking and integrity issues are found within **60 days**, so going past 60 is "may not be strong" |
| Backup retention window | Google "draw[s] the line between **30 and 90 days** of backups for many services" |
| Tier 1 backups | frequent, near the live datastore. Retained **hours to single-digit days**, restore in **minutes** |
| Tier 2 backups | distributed FS local to the site. Retained **single-digit or low double-digit days**, restore in **hours**. If you release twice a week, keep them "at least a week or two" |
| 1T vs 1E | 700 PB at 300 MB/s (SATA 2.0) takes about **8 decades** for one validation pass. Restore is "almost a full century" |
| Gmail validators | detect inconsistencies **within 24 hours**, and the storage code changes more often than once a week. One refactor cut disk-spindle contention by **60%**. The largest validator is split into **10–14 shards**, one shard per day |
| Deletion-rate alert (fn 133) | alert on the **global** rate across users when it crosses an extreme threshold "such as **10x the observed 95th percentile**", not per-user rates |
| Google Music 2012 | about **600,000** audio refs wrongly deleted, affecting **21,000** users. **436,223** were on tape and about **161,000** were never backed up. **1.5 PB** across **5,475** restore jobs and 5,000+ tapes. Restored in "just short of 7 days" (5 of them actual recovery) |

## Recommended practices & procedures

- **Size backups by recovery needs** (§ Second Layer): the recovery scenarios decide the method, the frequency of fulls and incrementals, the location and the retention. "Backups don't matter; what matters is recovery." Use fewer acceptable data-loss minutes to justify more incrementals. Take fulls off-peak and incrementals when busy. A faster recovery target calls for more local backups.
- **Use a tiered backup strategy.** Trade point-in-time recovery ("time-travel" inside Google) against tiering if you must, "but don't resort to not using either."
- **Soft-delete summary:** the trash folder is the primary defense against user error. Soft deletion is the primary defense against developer error and the secondary one against user error. Lazy deletion is the primary defense against internal developer error and the secondary one against external developer error. Apply soft or lazy deletion to revision history too. Build undelete APIs into platforms and **enable them by default** (Blobstore: default backups, end-to-end checksums, tombstone lifetimes).
- **Redundancy for backups:** stagger backups across independently failing sites. Use RAID, Reed-Solomon or GFS-style replication. Pick a *popular, continually used* scheme.
- **Scale with trust points:** verify data once it is immutable, then back up incrementally. Shard so that N tasks each handle 1/N of the data, with balanced and independent shards and no contention.
- **Out-of-band validation** (§ Third Layer: Early Detection): run MapReduce/Hadoop validation pipelines. "Only validate invariants that cause devastation to users." Use tiered validation: run cheaper daily validators that catch disasters within 24h, plus more rigorous checks less often. Validators can also auto-fix, as with Drive in 2013.
- **Org model:** a central infrastructure team provides the validation framework, and product teams own the business-logic validators.
- **Prove recovery** (§ Knowing That Data Recovery Will Work): test recovery continuously and automatically, from end to end. Alert when a recovery process fails to give a heartbeat that it succeeded. Use DiRT exercises.

## Anti-patterns

- "We have something even better than a backup—replication!"
- Treating archives as disaster-recovery backups.
- Manual, staged recovery tests.
- Trusting a storage API's consistency (Paxos/Megastore) without verifying it. "Perfect algorithms may not have perfect implementations."
- Validators so strict that engineers abandon them, or so loose that corruption slips through.
- Developers unfamiliar with the code changing deletion pipelines, especially batch ones. These caused the most devastating acute deletions.
- Per-user deletion-rate alerts. Use global aggregate alerts instead.
- Data you "can re-create" that becomes irreplaceable without anyone revisiting backups (Shakespeare annotations).
- Pipeline stages that assume timing ("three hours apart") and become race conditions as data grows (the Google Music root cause).

## Checklists

**Effective out-of-band validation demands:** validation job management; monitoring, alerts and dashboards; rate-limiting; troubleshooting tools; production playbooks; data validation APIs that make validators easy to add and refactor.

**Recovery-plan confirmation:**
- Are backups valid and complete, or are they empty?
- Are there enough machine resources for setup, restore and post-processing?
- Does recovery finish in reasonable wall time?
- Can you monitor recovery progress?
- Are you free of critical external dependencies (for example an offsite vault that isn't available 24/7)?

## Verbatim quotes

- "No one really *wants* to make backups; what people *really* want are *restores*."
- "Replication and redundancy are not recoverability."
- "You only know that you can recover your recent state if you actually do so."
- "Trust storage systems, but *verify*!"
- "Even the best armor is useless if you don't put it on."
- Conclusion: shrink recovery time N "with the goal of approaching N = 0", then shift from planning recovery to planning prevention: "all the data, all the time."
