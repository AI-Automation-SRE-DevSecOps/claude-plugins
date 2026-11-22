# Chapter 2: The Production Environment at Google, from the Viewpoint of an SRE

Source: https://sre.google/sre-book/production-environment/ (Site Reliability Engineering, O'Reilly 2017; written by JC van Winkel, edited by Betsy Beyer)

This chapter is mostly descriptive. It introduces terms used through the rest of the book and closes with a worked capacity example (the "Shakespeare" service). It has few explicit "do/don't" rules; the prescriptive content below is either stated in the text or implied directly by it.

## Core principles

- **Machine vs. server ("Hardware").** *Machine* = "A piece of hardware (or perhaps a VM)"; *server* = "A piece of software that implements a service." Machines are not dedicated to specific server programs; the cluster OS (Borg) allocates resources.
- **Homogeneous hardware.** Compute hardware in a Google-designed datacenter is "the same across the board" (footnote 9 admits multiple hardware generations exist).
- **Abstract hardware failure away with software ("System Software That 'Organizes' the Hardware").** Failure is routine: in one cluster in a typical year, "thousands of machines fail and thousands of hard disks break". Users and service teams should not have to deal with it.
- **Indirection instead of fixed addresses ("Managing Machines").** Tasks move between machines, so clients use the Borg Naming Service (BNS) name rather than IP:port.
- **Spread work across failure domains.** Borg binpacks tasks but "won't run all of a job's tasks on the same rack", because the top-of-rack switch would be a single point of failure.
- **Centralized control with cheap hardware ("Networking").** OpenFlow-based SDN uses "dumb" switches plus a central (duplicated) controller that precomputes best paths.
- **Design for modularity ("Our Software Infrastructure").** Services talk over RPC (Stubby/gRPC), often even when a local subroutine call would do, so the code can later be moved into a separate server.

## Concrete numbers, thresholds, formulas

**Topology (Figure 2-1):** tens of machines per **rack**, racks in a **row**, one or more rows form a **cluster**, a **datacenter** building usually holds multiple clusters, and nearby buildings form a **campus**.

**Networking:** Jupiter is a Clos fabric of hundreds of Google-built switches acting as a virtual switch with tens of thousands of ports. At its largest it supports **1.3 Pbps bisection bandwidth**. The B4 backbone is SDN (OpenFlow) and serves "a modest number of sites" with elastic bandwidth allocation.

**Borg resources:** jobs state their needs (e.g., "3 CPU cores, 2 GiB of RAM"). A task that goes over its request is killed and restarted.

**BNS path format:** `/bns/<cluster>/<user>/<job name>/<task number>`, which resolves to `<IP address>:<port>`.

**Protocol buffers vs. XML:** "3 to 10 times smaller, 20 to 100 times faster, and less ambiguous."

**Chubby example:** five replicas of a job, only one of which may do the work. Chubby (Paxos) picks which one.

**Request latency:** the whole Shakespeare request chain takes "just a few hundred milliseconds."

**Shakespeare capacity worked example ("Job and Data Organization"):**

| Item | Value |
|---|---|
| Per-task capacity (from load testing) | ~100 QPS |
| Expected peak (from trials with limited users) | ~3,470 QPS, so at least 35 tasks |
| Required with redundancy | **37 tasks = N+2** |
| Why +2 | During an update one task is down (36 left). A machine failure during the update leaves 35, "just enough to serve peak load" |

Regional peak split: NA 1,430 / South America 290 / Europe+Africa 1,400 / Asia+Australia 350 QPS (sum = 3,470).

| Region | Tasks | Redundancy |
|---|---|---|
| USA | 17 | N+2 |
| Europe | 16 | N+2 |
| Asia | 6 | N+2 |
| South America | **4 (not 5)** | **N+1**, which saves 20% of that region's hardware |

In South America, the team accepts "a small risk of higher latency" because GSLB can send overflow to another continent. In the larger regions, tasks are spread over two or three clusters. Footnote 13: N+2 assumes two simultaneous task failures are negligible. SPOFs such as top-of-rack switches or power distribution "may make this assumption invalid in other environments."

## Recommended practices and procedures

- **Capacity sizing (template):** N = ceil(peak QPS / per-task QPS). Add +1 for a rolling update and +1 for a concurrent unplanned failure, giving N+2. Size **per region** from the regional peak, not only from the global total. Drop to N+1 only where cross-region failover is acceptable.
- **Replicate data near compute.** Bigtable is replicated in each region for resilience and lower latency. Its eventual consistency is acceptable because the data changes rarely.
- **GSLB balances load at three levels:** geographic DNS, user service level (e.g., YouTube, Maps), and RPC level. Service owners provide a symbolic service name, a list of BNS addresses, and the capacity at each location (typically in QPS).
- **Store data that must be consistent in Chubby.** For example, BNS keeps its path-to-IP:port mappings there.
- **Use monitoring (Borgmon) for three things:** alerting on acute problems, comparing behavior (e.g., "did a software update make the server faster?"), and tracking resource use over time for capacity planning.
- **Make every server diagnosable.** Each server runs an HTTP server that exposes diagnostics and statistics.
- **Development workflow ("Our Development Environment"):** one shared repository; engineers may fix other teams' components and send a CL to the owner; "All software is reviewed before being submitted"; every CL triggers tests on everything that depends on it, directly or indirectly; some projects use **push-on-green** (automatic push to production once tests pass).
- **Reliability depends on** "rigorous testing and careful rollout" plus proactive error recovery such as graceful degradation.

## Anti-patterns (implied or stated)

- Dedicating specific machines to specific servers ("There's no specific machine that runs our mail server").
- Addressing tasks by raw IP:port when they move around.
- Putting all of a job's tasks in one failure domain (rack).
- Sizing capacity to exactly the peak (N) with no headroom for updates and failures.
- Serving a region from distant storage (for example, Asia backends reading a US Bigtable) and paying a large latency cost.
- Assuming N+2 is enough without checking for shared SPOFs (footnote 13).
- Note: Borg prefers restarting over-limit tasks, "as a slowly crashlooping task is usually preferable to a task that hasn't been restarted at all."

## Checklist (derived from the Shakespeare example)

1. Load-test to find per-task QPS.
2. Estimate peak QPS, broken down by region.
3. Compute N per region and add +2 (update + failure). Justify any N+1.
4. Spread tasks across clusters and racks. Check for shared SPOFs.
5. Replicate the data store per region, and confirm the consistency model fits how often the data changes.
6. Register with GSLB using the service name, BNS addresses, and per-location capacity.
7. Set up monitoring for alerting, before/after comparison, and capacity trends.

## Short verbatim quotes

- "Machines can run any server, so we don't dedicate specific machines to specific server programs."
- "In a single cluster in a typical year, thousands of machines fail and thousands of hard disks break"
- "a slowly crashlooping task is usually preferable to a task that hasn't been restarted at all."
- "we need at least 37 tasks in the job, or N+2"
- "All software is reviewed before being submitted."
- "In traditional terms, the frontend is called the client and the backend is called the server."
