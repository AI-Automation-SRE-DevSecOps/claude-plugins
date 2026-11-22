# Chapter 25 — Data Processing Pipelines (Google SRE Book)

Source: https://sre.google/sre-book/data-processing-pipelines/ — written by Dan Dennison, edited by Tim Harvey.

## Core principles

- **Definitions** (§Origin of the Pipeline Design Pattern; §Initial Effect of Big Data): a *data pipeline* is a program that reads data, transforms it, and outputs new data, typically run by a periodic scheduler such as cron. A "simple, one-phase pipeline" does periodic or continuous transformations on Big Data. A *multiphase pipeline* chains programs so each one's output feeds the next. **Depth** is the number of programs chained in series: a shallow pipeline can have depth 1, and a deep one "in the tens or hundreds of programs."
- Multiphase chaining is "typically designed for ease of reasoning about the system and not usually geared toward operational efficiency."
- **Periodic pipelines are fragile under growth** (§Challenges with the Periodic Pipeline Pattern). They are stable when there are enough workers and demand stays within capacity, and when the number of chained jobs and the relative throughput between jobs stay uniform. Once tuned, they start out reliable, but "organic growth and change inevitably begin to stress the system." Typical symptoms: jobs that miss their run deadline, resource exhaustion, and hanging chunks.
- **The frequency continuum has a floor** (§Drawbacks of Periodic Pipelines in Distributed Environments). Periodic pipelines run as lower-priority batch jobs on Borg, so they can face "open-ended startup delays." Scheduling delay sets a lower bound on achievable latency. Pushing the interval below that bound brings "undesirable behavior rather than increased progress."
- **Continuous processing with strong guarantees** (Workflow) is the recommended alternative for work that is continuous or will become continuous (§Summary).

## Concrete numbers, thresholds, formulas

- **Figure 25-1**: for a **~20-minute job**, "lowering the execution interval much below **40 minutes**" risks overlapping executions. The figure plots execution interval against idle time on a log scale.
- "Delays of up to **a few hours** might well be acceptable for pipelines that run daily."
- **Cost relationship**: "Execution cost is inversely proportional to requested startup delay, and directly proportional to resources consumed."
- **Figures 25-2/25-3 (Moiré)**: the peak that causes on-call pain comes when aggregate stacked load from three pipelines "nears **1.2M**".
- Thundering herd: "potentially **thousands** of workers immediately start work"; the worst case is "a buggy **10,000** worker pipeline job."
- Workflow was developed in **2003**.
- Workflow has **four** correctness guarantees. The text also counts its build-up as "double" (lease + unique filenames) and then "triple" (+ configuration task ID) before adding the server token.

## Failure modes / anti-patterns Google warns against

1. **Uneven work distribution / hanging chunks** (§Trouble Caused By Uneven Work Distribution). Work is split at a point of indivisibility, such as the customer, so end-to-end runtime is "capped to the runtime of the largest customer." Stages wait for the whole computation (for example, a sort) before moving on, so completion is blocked on the worst-case chunk.
2. **Kill-and-restart a hanging job** is the "sensible" or "default" response, but it makes things worse. Pipelines usually lack checkpointing, so all chunks restart from the beginning and the earlier time, CPU, and human effort are wasted.
3. **Shortening the interval below the scheduling-delay floor.** Runs can stack up on the scheduler, or a nearly finished run can be killed when the next one starts, "completely halting all progress."
4. **Overusing the batch scheduler.** This risks preemptions when cluster load is high. Running a periodic pipeline well is "a delicate balance between high resource cost and risk of preemptions."
5. **Metrics reported only at completion** (§Monitoring Problems). "If the job fails during execution, no statistics are provided." For long-running pipelines, real-time metrics can matter more than overall metrics because emergency response depends on them.
6. **Thundering herd** (§"Thundering Herd" Problems). Too many workers, misconfiguration, or faulty retry logic can overwhelm servers, shared cluster services, and the network. With *no* retry logic, dropped work causes correctness problems. With *naive* retry logic, retries compound the overload. Inexperienced engineers add workers when a job runs late, which amplifies the problem.
7. **Moiré load pattern** (§Moiré Load Pattern). Two or more pipelines whose executions occasionally overlap hit a shared resource at the same time. It is hard to see in isolation and shows up most clearly in stacked plots of shared-resource usage. It can affect continuous pipelines too, but is rarer when load arrives evenly.
8. **Avoiding resource acquisition.** Teams are reluctant to contribute capacity to a shared pool. The fix is to secure enough capacity and to distinguish batch scheduling resources from production priority resources so that acquisition costs can be justified.
9. **Putting a load balancer in front of independent Task Masters**, an example misconfiguration that the server token guards against.

## Recommended practices & procedures

- **Up-front scoping** (§Introduction to Google Workflow): when designing a system that involves a data pipeline, scope the "expected growth trajectory, demand for design modifications, expected additional resources, and expected latency requirements from the business." Business demand for continuous results tends to arrive at the least convenient time to refactor.
- **Workflow architecture** (§Workflow as Model-View-Controller Pattern). It combines the leader-follower (workers) pattern with system prevalence, which gives exactly-once semantics.
  - *Model* = **Task Master**. It holds all job state in memory and synchronously journals mutations to persistent disk. Best performance comes from storing only **pointers to work** in the Task Master, with the actual data in a common filesystem or other storage.
  - *View* = **workers**. They are completely stateless, can be discarded at any time, and update state transactionally with the master.
  - *Controller* (optional) handles auxiliary activities: runtime scaling, snapshotting, workcycle state control, rolling back pipeline state, and global interdiction for business continuity.
- **Stages** (§Stages of Execution in Workflow): depth is set by task groups in the Task Master, one per stage. Each stage has a worker type, and workers can self-schedule by choosing which type of work to perform. All work is executed, "or at least reflected in permanent state, exactly once."
- **Why not just use Spanner or a database?** Workflow tasks are unique and immutable, and even a lease change creates a new task. A database whose transaction log acts as the journal would need every read to be part of a long-running transaction, which is "terribly inefficient."
- **Business continuity** (§Ensuring Business Continuity):
  - Task Master journals are stored on **Spanner**, which serves as a globally consistent but low-throughput filesystem.
  - The writer Task Master is elected through **Chubby**, and the result is persisted in Spanner. Clients find the current Task Master through internal naming services.
  - Two or more local Workflows run in distinct clusters, plus **reference tasks** in a global Workflow. A "stage 1" helper inserts reference tasks, and "stage n" removes them transactionally when tasks finish. If the global Workflow is unavailable, the local Workflow **blocks**, which preserves correctness.
  - The "stage 1" helper (the controller) updates a **heartbeat task**. If the heartbeat isn't updated within the timeout, the remote Workflow's helper takes over the in-progress work recorded in the reference tasks.

## Checklist: the four Workflow correctness guarantees (verbatim list, §Workflow Correctness Guarantees)

1. "Worker output through configuration tasks creates barriers on which to predicate work."
2. "All work committed requires a currently valid lease held by the worker."
3. "Output files are uniquely named by the workers."
4. "The client and server validate the Task Master itself by checking a server token on every operation."

Mechanics: an orphaned worker can keep writing its own uniquely named file but cannot commit without the lease, so it can't destroy valid output. All tasks are versioned, and any update or lease change produces a new task ID. A commit must reference the configuration task ID that was used, so after a config change, in-flight work from old leases is thrown away.

## Short verbatim quotes

- "the collective SRE experience has been that the periodic pipeline model is fragile."
- "nothing is harder on cluster infrastructure and the SREs responsible for a cluster's various services than a buggy 10,000 worker pipeline job."
- "Periodic pipelines shouldn't have inherent monitoring problems, but we have observed a strong association."
- "if a data processing problem is continuous or will organically grow to become continuous, don't use a periodic pipeline."
