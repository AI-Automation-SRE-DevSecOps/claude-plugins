# SRE Book, Chapter 22: Addressing Cascading Failures (Mike Ulrich)

Source: https://sre.google/sre-book/addressing-cascading-failures/

## Core principles
- **Definition:** "A cascading failure is a failure that grows over time as a result of positive feedback."
- **Overload is the most common cause** (*Server Overload*). The spread across clusters can take "on the order of a couple minutes".
- **Resource exhaustion feeds itself** (CPU, memory, threads, file descriptors; *Dependencies among resources*). Secondary symptoms look like the root cause.
- **Recovery takes far less load than the failure point** (*Service Unavailability*). Crash-looping servers are hit with traffic as soon as they come back.
- **When overloaded, something must give.** It is better to allow some errors or degraded results than to try to serve every request (*Closing Remarks*).
- Steady-state "improvements" can raise the risk of a full outage. Retries, shifting load away from unhealthy servers, killing unhealthy servers and adding caches are all examples (*Closing Remarks*).

## Concrete numbers, thresholds and formulas (exact)
| Topic (section) | As written |
|---|---|
| Overload example (*Server Overload*) | Cluster A handles 1,000 QPS. When B fails, A gets 1,200 QPS, and successful requests drop "well below 1,000 QPS" |
| Recovery (*Service Unavailability*) | Healthy at 10,000 QPS, cascade at 11,000 QPS. Dropping to 9,000 QPS "will almost certainly not stop the crashes". If 10% of servers are healthy, load must drop to about 1,000 QPS |
| Capacity planning (*Preventing Server Overload*) | Breaking point 5,000 QPS per cluster, peak 19,000 QPS, load spread evenly, gives "approximately six clusters" at **N + 2**. Footnote 109: even spread is "often not a good assumption due to geography" |
| Queue latency (*Queue Management*) | Queue size 10x threads, 100 ms per request. With a full queue, a request takes **1.1 s** |
| Queue sizing (*Queue Management*) | For steady traffic, keep queues small relative to the thread pool, "e.g., 50% or less". Gmail often uses queueless servers |
| Shedding (*Load Shedding*) | Return **HTTP 503** when in-flight client requests exceed a set number |
| Naive retry (*Retries*) | 5 s timeout, up to 10 attempts. Backend limit 10,000 QPS/task, sent 10,100 QPS. Retries every 1,000 ms grow 100, then 200, then 300 QPS... Dropping to 9,000 QPS may not fix it |
| Retry budget (*Retries*) | Example: "only allow 60 retries per minute in a process" |
| Retry amplification (*Retries*) | 3 retries (4 attempts) at each of 3 layers (backend, frontend, JS) gives **64 attempts (4^3)** on the DB. Total attempts equal the *product* of attempts per layer |
| Deadline propagation (*Deadline propagation*) | A picks 30 s and spends 7 s, so A to B gets 23 s. B spends 4 s, so B to C gets 19 s. Counterexample: 10 s deadline, B spends 8 s, so it should send 2 s but hardcodes 20 s. C dequeues at 5 s and thinks it has 15 s left. Trim outgoing deadlines by "a few hundred milliseconds" |
| Bimodal latency (*Bimodal latency*) | 10 servers x 100 threads = 1,000 threads. 1,000 QPS x 0.1 s = 100 threads busy. If 5% of requests never complete and the deadline is 100 s, those need 5,000 threads (50 QPS x 100 s). Only **19.6%** of requests get served (1,000 / (5,000 + 95)), an **80.4% error rate** |
| Per-client fairness (*Bimodal latency*) | Allow any one client to occupy at most **25% of threads** |
| Drop traffic (*Drop Traffic*) | If the whole service is crash-looping, "only allow, say, 1% of the traffic through" |

## Recommended practices and procedures
**Preventing Server Overload** (in "rough priority order"):
1. Load test capacity limits *and* the failure mode under overload. This is "the most important exercise".
2. Serve degraded results.
3. Make servers reject requests when overloaded. Fail early and cheaply.
4. Make higher-level systems reject requests (reverse proxies, load balancers, individual tasks). Rate limiting often ignores overall service health.
5. Do capacity planning, paired with performance testing. It is necessary but not sufficient.

**Shedding:** Shed per task by CPU, memory or queue length. Use LIFO or CoDel instead of FIFO, with deadline propagation. Graceful degradation reduces the work itself (in-memory subset, cheaper ranking).

**Retries:** Use randomized exponential backoff. Cap retries per request. Use a server-wide retry budget. Retry at one layer. Separate retriable from non-retriable errors. Return a specific "overloaded" status so clients don't retry.

**Deadlines:** Always set one. Check the remaining time before each stage. Propagate absolute deadlines and cancellations (hedged requests [Dea13]). Bound outgoing deadlines, but know your traffic mix. Look at latency *distributions*, not means. Use fail-fast.

**Caches** (*Slow Startup and Cold Caching*): With a **latency cache**, the service can carry expected load with an empty cache. With a **capacity cache**, it cannot. New caches must be latency caches or engineered to be safe as capacity caches. Ramp load slowly into cold clusters.

**Always Go Downward in the Stack:** Avoid intra-layer calls in the request path, because they risk distributed deadlock. The backend should tell the frontend to retry on the correct backend instead of proxying.

**Triggers:** process death (Query of Death), updates (keep headroom, push off-peak), rollouts (check recent changes and revert), organic growth, drains, request profile changes, resource limits.

**Immediate Steps:** increase resources; stop health-check deaths; restart servers; drop traffic; degrade; eliminate batch load and bad traffic.

## Anti-patterns Google warns against
- Naive retry loops (the Go example has no backoff or budget). Retrying at several layers.
- No deadline, or one far too high: "several orders of magnitude longer than the mean request latency" (the example is 3 orders).
- Hardcoding downstream deadlines instead of propagating them.
- Load balancing that avoids servers which returned errors, which feeds the snowball.
- Relying on slack CPU from overcommitted machines.
- Caches that quietly become hard dependencies.
- Degradation code that is never exercised. Run a few servers near overload, alert when many servers degrade, and keep a quick off-switch.
- Mixing up health checks. **Process** health ("responding *at all*?") is for the scheduler. **Service** health ("able to respond to *this class of requests*?") is for the load balancer.
- Restarting servers blindly. Find the source first and canary the restart, because with a cold cache a restart amplifies the outage.

## Templates and checklists
**Questions for load shedding and degradation:** Which metrics trigger it (CPU, latency, queue length, threads; automatic or manual)? What actions happen in degraded mode? At what layer, every layer or one choke-point?

**Load tests** (*Testing for Cascading Failures*): break each component; test gradual and impulse load; test recovery (Does it exit degraded mode unaided? How far must load drop?). Production tests: cut tasks, lose a cluster, blackhole backends. Check that clients back off. Test noncritical backends that are down *and* blackholing.

**Drop Traffic procedure:** (1) address the trigger, for example by adding capacity; (2) cut load until crashing stops (~1%); (3) let most servers become healthy; (4) ramp up gradually. Drop less important traffic first. Fix the root cause first, or the cascade returns.

**Restart when:** GC death spiral, requests without deadlines blocking threads, or deadlock.

## Short verbatim quotes
- "If at first you don't succeed, back off exponentially." (Dan Sandler)
- "Why do people always forget that you need to add a little jitter?" (Ade Oshineye)
- "you don't get credit for late assignments with RPCs."
- "Remember that the code path you never use is the code path that (often) doesn't work."
- "Be careful when evaluating changes to ensure that one outage is not being traded for another."
