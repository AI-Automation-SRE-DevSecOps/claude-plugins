# Load Balancing, Overload and Cascading Failures

Sources: SRE Book ch.19 "Load Balancing at the Frontend", ch.20 "Load Balancing in the Datacenter", ch.21 "Handling Overload", ch.22 "Addressing Cascading Failures", ch.27 (client behavior). SRE Workbook ch.11 "Managing Load".

## Principles

- **Overload is inevitable.** Plan for it in this order: redirect the load, serve **degraded** responses, then reject what you can't serve, cheaply. (Book ch.21)
- **Model capacity in resources (CPU-seconds and so on), not QPS.** The cost of a request changes over time, so "a moving target makes a poor metric". (Book ch.21)
- **Every task must protect itself.** Load-balancing state propagates imperfectly.
- **An overloaded backend keeps serving what it can and rejects the rest gracefully.** It never stops taking all traffic. A task should keep serving its provisioned load, without crashing, at **2× to 10×** overload. (Book ch.21)
- **Recovery needs much less load than the point where failure began.** A service that cascades at 11,000 QPS won't recover at 9,000. If only 10% of servers are healthy, load must drop to about 1,000 QPS. (Book ch.22)
- "A cascading failure is a failure that grows over time as a result of positive feedback."

## Retries

| Rule | Value | Source |
|---|---|---|
| Per-request budget | **At most 3 attempts** (attempt counter 0 to 2, carried in request metadata) | Book ch.21 |
| Per-client budget | Retry only while retries are **below 10%** of requests. Worst-case load is then about **1.1×** instead of nearly 3× | Book ch.21 |
| Process-wide budget | For example, "only allow 60 retries per minute in a process" | Book ch.22 |
| Backoff | **Randomized exponential backoff with jitter.** "If at first you don't succeed, back off exponentially" | Book ch.22, App. B |
| Layer | **Retry only at the layer immediately above the one rejecting.** Three retries at each of three layers gives 4³ = **64×** load on the database | Book ch.21, ch.22 |
| Signal | Return an explicit **"overloaded; don't retry"** error once many retries are seen, and don't retry non-retriable errors (most 4xx) | Book ch.21, ch.27 |

Clients must also **add jitter to scheduled and periodic requests**. Syncing every 60 s instead of 600 s is 10× the load (Book ch.27). Pokémon GO's synchronized retries hit **20× the previous peak**. The fix was truncated exponential backoff with jitter. (Workbook ch.11)

```python
import random, time

def call_with_retries(fn, *, max_attempts=3, base=0.1, cap=10.0, retry_budget):
    """retry_budget: shared token bucket that allows retries up to ~10% of request volume."""
    for attempt in range(max_attempts):
        try:
            return fn(attempt=attempt)            # pass the attempt number in request metadata
        except OverloadedDontRetry:
            raise                                 # the server asked us not to retry
        except RetriableError:
            if attempt == max_attempts - 1 or not retry_budget.try_acquire():
                raise
            time.sleep(random.uniform(0, min(cap, base * 2 ** attempt)))   # full jitter
```

## Deadlines

- **Always set a deadline.** RPC defaults are often effectively unbounded. (Workbook ch.11)
- **Propagate absolute deadlines.** A 30 s deadline with 7 s spent leaves 23 s downstream. Trim a few hundred milliseconds for network time. Never hardcode a downstream deadline larger than what remains. (Book ch.22)
- **Check the remaining time before each stage**, and drop work whose deadline has passed: "you don't get credit for late assignments with RPCs."
- **Propagate cancellations** too.
- Avoid deadlines **orders of magnitude longer than the mean latency**. Bimodal latency is the trap: if 5% of requests hang until a 100 s deadline, only **19.6%** of requests get served.
- Limit any one client to about **25% of threads**, so a slow client can't take them all. (Book ch.22)

## Load shedding and graceful degradation

- **Shed per task** based on CPU, memory, queue length or in-flight requests. Return a fast **HTTP 503** (or RESOURCE_EXHAUSTED) once in-flight requests exceed a limit. (Book ch.22)
- **Queues**: keep them small, at most about **50% of the thread pool** for steady traffic. Prefer **LIFO or CoDel** over FIFO, combined with deadline checks. A full queue of 10× threads at 100 ms per request means each request waits 1.1 s.
- **Criticality** (Book ch.21): four levels, set close to the client and propagated through every RPC:

| Level | Meaning |
|---|---|
| CRITICAL_PLUS | Most critical; failure has serious user-visible impact |
| CRITICAL | **Default for production traffic** |
| SHEDDABLE_PLUS | Partial unavailability expected; **default for batch jobs** |
| SHEDDABLE | Frequent partial and occasional full unavailability are acceptable |

Shed lower criticalities first. Provision for all CRITICAL and CRITICAL_PLUS traffic.

- **Client-side adaptive throttling** (Book ch.21): each client tracks `requests` and `accepts` over the last **2 minutes**, and rejects locally with probability `max(0, (requests − K·accepts) / (requests + 1))`. **K = 2** is preferred; a lower K throttles harder.
- **Per-customer quotas** in CPU-seconds per second. Quotas may add up to more than total capacity, because customers rarely all peak at once. A global overload should hurt only the customers who are over quota.
- **Degraded mode** means cheaper work: search a smaller corpus, use a stale cache, skip optional features. **Exercise degradation regularly**: "the code path you never use is the code path that (often) doesn't work". Alert when many servers degrade, and keep a quick off-switch.
- **Batch traffic**: route large batch clients through a **batch proxy** that "acts like a fuse".
- **Make load shedding and load balancing cooperate.** Count shed requests as more than 100% utilization (Dressy counted them as 120% CPU) so the load balancer moves traffic away. Autoscale **before** shedding. (Workbook ch.11)

## Load balancing

**Frontend (Book ch.19):**
- DNS load balancing alone isn't enough. Use **DNS plus a VIP/anycast** layer.
- "Best location" means close, with enough capacity, and healthy, not simply the nearest.
- Keep DNS TTLs low, but some resolvers ignore them. EDNS0 client-subnet helps with location.
- Use **consistent hashing** or connection tracking in network load balancers. `hash mod N` reshuffles almost every connection when a backend changes.

**Datacenter (Book ch.20):**
- **Lame-duck state**: a backend that is shutting down tells clients to stop sending new requests, finishes the ones in flight, then exits. A typical drain is 10–150 s.
- **Health checks** come in two kinds. **Process health** ("responding at all?") is for the scheduler. **Service health** ("can it serve this class of request?") is for the load balancer. Don't mix them up. (Book ch.22)
- **Subsetting**: each client connects to a subset of backends, typically **20–100**. Use **deterministic subsetting** with a new shuffle each round. Random subsetting spreads load unevenly.
- **Weighted round robin** works best: backends report utilization, and clients weight by it and penalize errors. Least-loaded round robin can **sinkhole** traffic to a backend that fails fast, unless recent errors count as active load.

**Autoscaling (Workbook ch.11):**
- Set min/max bounds and keep a minimum number of instances per location.
- Provide a **kill switch** that is "easy, obvious, fast, and well documented".
- Analyze dependencies first: scaling up can overload a backend downstream.
- Watch for bad signals, such as scaling on CPU that is being spent on rejections.

## Caches and cold starts (Book ch.22)

- A **latency cache** only speeds things up; the service can carry its load with the cache empty.
- With a **capacity cache**, it can't. A cold or flushed capacity cache means instant overload.
- Make sure new caches are latency caches, or treat a capacity cache as a hard dependency with its own SLO. **Ramp load slowly into cold clusters.**
- **Always call downward in the stack.** Avoid calls between tasks in the same layer on the request path, which can cause distributed deadlock.

## Cascading failure triggers (Book ch.22)

Process death (a "query of death"), updates and rollouts (no spare capacity during a push), organic growth, planned drains, a change in request mix, and hitting resource limits.

Resource exhaustion feeds on itself: CPU starvation slows requests, which hold more memory and threads, which slows GC, which uses more CPU. The secondary symptoms look like the root cause.

## Responding to a cascade (Book ch.22)

1. **Add resources** if you can. Fix the trigger, for example by rolling back the bad change.
2. **Stop health-check deaths**: don't let the scheduler kill tasks that are only slow.
3. **Drop traffic hard** (admit about 1%), let servers become healthy, then **ramp up gradually**. Drop the least important traffic first.
4. **Enter degraded modes**, and **remove batch and bad traffic**.
5. **Restart only when it helps**: GC death spirals, requests stuck without deadlines, deadlocks. Canary the restart first, since restarting into a cold cache can make things worse.

## Prevention (Book ch.22, in priority order)

1. **Load-test to the breaking point and beyond**, and test recovery: does the service leave degraded mode on its own, and how far must load drop? This is "the most important exercise".
2. Serve degraded results.
3. Reject requests early and cheaply when overloaded.
4. Reject at higher layers (reverse proxies, load balancers) based on the health of the whole service, not just a flat rate limit.
5. Capacity-plan at N+2 (see `capacity-launch.md`). It's necessary but not sufficient.

Also run production tests: kill tasks, lose a cluster, blackhole a backend, and check that clients back off. "Be careful when evaluating changes to ensure that one outage is not being traded for another." Retries, shifting load away from unhealthy servers, and caches can all make a full outage more likely.

## Checklist

- [ ] Capacity and quotas defined in resources, per customer and per criticality
- [ ] Retries: ≤ 3 attempts, < 10% per client, backoff with jitter, one layer only, a "don't retry" signal
- [ ] Deadlines set everywhere and propagated, checked before each stage; cancellations propagated
- [ ] Per-task load shedding by criticality; LIFO or CoDel queues of ≤ 50% of the thread pool
- [ ] Client-side adaptive throttling (K = 2); jitter on periodic client requests
- [ ] Graceful degradation exercised regularly, with an off-switch
- [ ] Process and service health checks separated; lame-duck draining
- [ ] Caches classified as latency or capacity; a cold-start ramp plan
- [ ] Load tests beyond the breaking point, including recovery behavior
- [ ] A cascade playbook: drop to about 1%, recover, ramp up
