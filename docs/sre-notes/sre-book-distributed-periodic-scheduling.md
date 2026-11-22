# Chapter 24 — Distributed Periodic Scheduling with Cron

Source: https://sre.google/sre-book/distributed-periodic-scheduling/ (Site Reliability Engineering, 2017). Written by Štěpán Davidovič, edited by Kavita Guliani. Parts were previously published in ACM Queue (March 2015, vol. 13, issue 3).

## Core principles

- **Skip a launch rather than risk a double launch ("fail closed").** (Cron Jobs and Idempotency) Jobs differ. Some are idempotent (garbage collection), some must never run twice (an email newsletter), and some must never be skipped (monthly payroll). Google's default is to skip "as much as the infrastructure allows", because a skipped launch is easier to recover from than a double launch.
- **Job owners monitor their own cron jobs.** The owner can have the cron service expose state for its jobs, or set up independent monitoring of what the jobs do. The owner then deals with a skipped launch in whatever way suits the job.
- **Decouple processes from machines.** (Extended Infrastructure) Cron on one machine is not acceptable. In a datacenter of exactly 1,000 machines, losing 1/1000th of them could take out the whole cron service. A datacenter scheduler such as Borg places replicas and handles machine deaths. "Launching a job" becomes sending one or more RPCs to that scheduler.
- **Base services with wide impact should have very few dependencies.** (Tracking the State of Cron Jobs) Cron should keep working for some time even if parts of the datacenter fail, and it should not depend on downstream systems that serve many internal users.
- **Strong consistency of launch state is essential**, because many jobs are not idempotent. The design is built on Paxos.
- **Deploy per datacenter, not globally.** A per-datacenter deployment has low latency and shares fate with the datacenter scheduler, which is cron's core dependency.

## Concrete numbers, thresholds and definitions (as written)

- The smallest crontab scheduling interval is **one minute**. Rescheduling a process to another machine can take longer than that.
- For a job that runs every five minutes, a **"one- to two-minute delay"** from rescheduling overhead is "potentially unacceptably substantial". The answer is **hot spares**.
- Paxos health checking detects that the leader has died **"within seconds"**. The target is to stay "well within a generally tolerable **one-minute failover time**". Leader election "must converge **faster than one minute**".
- **Three replicas** run by default, which means three copies of the Paxos logs on local disk.
- A traditional cron holds "on the order of **tens** of cron jobs". A datacenter-scale cron serves thousands of machines.
- Thundering herd example: **30 teams** each run a daily midnight job that spawns a MapReduce with thousands of workers.
- Midnight daily crontab: `"0 0 * * *"`. The `?` extension means the system picks the value by **hashing the cron job configuration over the given range** (for example `0..23` for hour).
- Snapshot example: 1,000 "increment counter by 1" log entries compact into one entry, "Set counter to 1,000."
- Paxos variant: **Fast Paxos [Lam06]**. Its internal leader replica also serves as the cron service leader.
- anacron re-launches missed jobs only if they run **daily or less frequently**. It does this by keeping a file of last-launch timestamps.

## Architecture and procedures

**Leader** (The Roles of the Leader and the Follower):
1. Keeps a list of jobs sorted by next launch time and waits for the first one.
2. Announces through Paxos, **synchronously**, that a launch is about to start. It does not launch until a Paxos quorum confirms. It also computes the next launch time and keeps the launch spec in sync with the followers.
3. Identifies each launch by **job + scheduled start time**. Without the start time, tracking becomes ambiguous, especially for jobs that run every minute.
4. Announces completion synchronously through Paxos. Completion is recorded whether the launch succeeded or failed (for example, if the datacenter scheduler was down). What gets tracked is that the launch was *attempted*.
5. When it loses leadership, it **must immediately stop talking to the datacenter scheduler**. Holding leadership guarantees mutual exclusion.

**Follower**: copies state from the leader through Paxos. When it learns a launch has started, it updates the job's next scheduled launch time synchronously. It tracks **open launches** (started but not completed). After a new leader is elected, every open launch must be concluded.

**Resolving partial failures**: each launch has two synchronization points, "about to perform" and "finished". To finish an open launch, at least one of these must hold:
- every operation on external systems that may need to be continued is **idempotent**, or
- the state of every external operation can be **looked up** to tell unambiguously whether it completed.

The recommended technique is to **precompute datacenter job names before launching**, so naming involves no mutating operations, distribute those names to all replicas, and **include the scheduled launch time in the names**. A new leader looks up the precomputed names and launches only the ones that are missing. Without the launch time in the name, an unusually long failover leads to a double launch: the new leader sees the job already completed and runs it again.

**Storing the state**: a Paxos log grows without limit, so it is compacted with **snapshots**. Logs and snapshots live on the replicas' local disks. **Snapshots are also backed up to a distributed filesystem** because they are the most critical state: lose them and you "start from zero". **Logs are deliberately not stored on the distributed filesystem**, because frequent small writes cost too much performance. Losing logs means only a bounded loss back to the last snapshot, and snapshots are taken at configurable intervals. A new replica can fetch the snapshot and logs from a running peer, so it does not depend on any local state.

**Thundering herd**: the `?` crontab extension spreads launches out. Load stays spiky anyway (Figure 24-3), often because some jobs depend on external events and must run at a specific time.

## Anti-patterns Google warns against

- Hosting the cron service on one machine.
- Keeping cron state in a general-purpose distributed filesystem such as GFS or HDFS. These are built for very large files, and small writes there are "very expensive and come with high latency".
- Launching before the quorum has acknowledged. On failover, followers might launch the same job again.
- Identifying a launch by job name alone, without the scheduled time.
- A deposed leader that keeps talking to the datacenter scheduler.
- Leaving it to user habit to schedule every daily job at midnight.
- Ignoring partial (multi-RPC) launch failures during recovery.

## Checklist (derived from the chapter)

- [ ] Classify each job: is it idempotent? can a launch be skipped? Default to fail closed.
- [ ] The owner monitors the job's launches and their effects.
- [ ] Replicas run under a datacenter scheduler, spread across power domains (no single PDU failure takes them all out).
- [ ] Paxos quorum write before each launch and after it, keyed by job + scheduled time.
- [ ] Leader failover is under 1 minute, and a deposed leader is fenced off from the scheduler.
- [ ] External launches are idempotent or can be looked up. Job names are precomputed and include the launch time.
- [ ] Snapshots are backed up off-host. Losing logs is an accepted, bounded risk.
- [ ] Hashed (`?`) scheduling is used to avoid a thundering herd.

## Short verbatim quotes

- "we favor skipping launches rather than risking double launches, as much as the infrastructure allows." (Cron Jobs and Idempotency)
- "we prefer to 'fail closed' to avoid systemically creating bad state." (Cron Jobs and Idempotency)
- "Base services for which outages have wide impact (such as cron) should have very few dependencies." (Tracking the State of Cron Jobs)
- "as soon as it loses its leadership for any reason, it must immediately stop interacting with the datacenter scheduler." (The leader)
- "Snapshots are in fact our most critical state" (Storing the State)
- "The election must converge faster than one minute" (The follower)
