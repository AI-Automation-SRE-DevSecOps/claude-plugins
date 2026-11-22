# SRE Book, Chapter 21: Handling Overload

Source: https://sre.google/sre-book/handling-overload/ (Alejandro Forero Cuervo, ed. Sarah Chavis). Companion to Ch. 20 (Load Balancing in the Datacenter); leads into Ch. 22 (Addressing Cascading Failures).

## Core principles

- **Overload is inevitable** (intro): no load balancing policy prevents it everywhere. Handle it in order: redirect when possible, serve **degraded responses** when necessary (search a small part of the corpus, use a stale local copy), and handle resource errors transparently when all else fails.
- **Model capacity in resources, not QPS** ("The Pitfalls of 'Queries per Second'"): per-query cost shifts over time. Request **cost** is normalized CPU time. CPU alone usually works (GC turns memory pressure into CPU; other resources can be provisioned to outlast CPU); otherwise account for each resource separately.
- **Global overload should hurt only misbehaving customers** ("Per-Customer Limits"). Provision for negotiated usage and enforce per-customer quotas.
- **Every task must protect itself** (Conclusions): load-balancing mechanisms depend on distributed state propagation, which is sometimes imperfect.
- **An overloaded backend should not stop accepting all traffic.** It should keep accepting as much as it can process and reject the rest gracefully.

## Concrete numbers, thresholds and formulas

- Datacenter cap example: 100 tasks × 500 req/s = no more than **50,000 QPS** sent to that datacenter.
- Per-customer quota example (backend with **10,000 CPUs** worldwide): Gmail 4,000; Calendar 4,000; Android 3,000; Google+ 2,000; every other user 500 **CPU seconds per second**. These **can add up to more than 10,000**, because it's unlikely all customers hit their limits at once.
- **Adaptive throttling** (Client-Side Throttling): each client tracks, over the **last two minutes**, `requests` (attempted by the application layer) and `accepts` (accepted by the backend). Client request rejection probability (Eq. 21-1, shown as an image in the source):

  `max(0, (requests − K × accepts) / (requests + 1))`

  - Self-regulation begins when requests = K × accepts. **K = 2 is preferred.**
  - Lower K means more aggressive throttling. With K = 1.1, the backend rejects only one request for every 10 accepted.
  - With K = 2, under heavy overload "backends end up rejecting one request for each request they actually process" (about half of backend resources go to rejections).
  - Locally rejected requests still count in `requests`. This is intentional: the drop probability rises as demand grows.
- **Retry limits** (Deciding to Retry):
  - Per-request retry budget: **up to three attempts**. The attempt counter in request metadata starts at 0 and increments up to **2**.
  - Per-client retry budget: retry only while retries are **below 10%** of requests.
  - Worst case with the per-request budget only: load grows to **just below 3X**. Adding the 10% per-client budget brings it to **about 1.1x** in the general case.
- **Figure 21-1** (attempt histograms per 1,000 initial requests, per-client budget ignored; counts for attempt 0 / 1 / 2):
  | Condition | Attempt 0 | Attempt 1 | Attempt 2 |
  |---|---|---|---|
  | No overload | 1000 | 0 | 0 |
  | Light (1% of tasks reject 10%) | 1000 | 1 | 0 |
  | Medium (5% of tasks reject 50%) | 1000 | 25 | ~1 |
  | Heavy (20% of tasks reject 90%) | 1000 | 180 | 32 |
- **Overload tolerance target** (Conclusions): a task should keep serving its provisioned rate without significant latency impact, and without crashing, up to **somewhere above 2x or even 10x** its provisioned traffic.

## Criticality (four values)

| Value | Meaning |
|---|---|
| CRITICAL_PLUS | Most critical; failure causes serious user-visible impact |
| CRITICAL | **Default for production jobs**; user-visible impact, less severe |
| SHEDDABLE_PLUS | Partial unavailability expected; **default for batch jobs** (they can retry minutes or hours later) |
| SHEDDABLE | Frequent partial and occasional full unavailability expected |

- Services must provision for all expected CRITICAL and CRITICAL_PLUS traffic.
- When a customer is out of quota, a backend rejects a given criticality only once it is already rejecting all lower criticalities. Per-customer limits can be set per criticality.
- An overloaded task rejects lower criticalities sooner. Adaptive throttling keeps separate stats for each criticality.
- Criticality is **orthogonal to latency requirements and QoS**. Example: search-as-you-type is highly sheddable but latency-sensitive.
- Criticality propagates automatically through RPCs (B and C inherit A's). Set it **as close to the browser or mobile client as possible** (HTTP frontends); override only where it makes sense. Four values suffice; more would cost more to operate.

## Utilization signals

- Utilization is usually CPU rate / CPUs reserved for the task (sometimes plus memory). Near thresholds, requests are rejected by criticality, with **higher thresholds for higher criticalities**.
- Signals are task-local. **Executor load average** counts running or runnable threads, smooths with exponential decay, and rejects when it exceeds the processors available to the task, so brief fan-out spikes are absorbed and sustained load is shed. Signals are pluggable (e.g., memory pressure) and combinable.

## Recommended practices and procedures

- Treat the two overload cases differently (Handling Overload Errors). If a **large subset** of tasks in a datacenter is overloaded, don't retry; bubble the error up to the caller. If only a **small subset** is overloaded (the typical case), retry immediately.
- Treat retries the same as new requests in load balancing. Don't force them onto a different task; reject quickly and let the retry land elsewhere, which acts as organic load balancing.
- Backends keep histograms of the retry counter. When those histograms show many retries, return **"overloaded; don't retry"** instead of "task overloaded."
- **Retry only at the layer immediately above the rejecting layer** (Figure 21-2: Frontend → Backend A → Backend B → DB Frontend). Upper layers either pass along "overloaded; don't retry" or return a degraded response.
- Reject out-of-quota requests quickly; add client-side throttling where rejecting costs nearly as much as serving (footnote 106: Doorman).
- Load from Connections: health checks on many low-rate clients can cost more than serving the requests. Check less often, or create and destroy connections dynamically. For connection bursts from large batch jobs, base cross-datacenter load balancing on cluster utilization, and require a **batch proxy** (batch client → batch proxy → backend) that "acts like a fuse."

## Anti-patterns Google warns against

- Modeling capacity as QPS or with static request features (e.g., keys read).
- Assuming rejection is free; CPU can go mostly to rejecting.
- Retrying at multiple layers (combinatorial explosion).
- Ad hoc, incompatible per-service criticality schemes.
- Ignoring degradation (OOM, thrashing, cascading failure).
- An overloaded backend refusing all traffic.
- Client-side throttling for sporadic clients (poor view of backend state).

## Checklist

- [ ] Capacity and quotas in CPU or per-resource, not QPS; quotas per customer and criticality
- [ ] Adaptive throttling: 2-minute window, K = 2, per-criticality stats
- [ ] Criticality set at the frontend and propagated
- [ ] Task-local shedding (executor load average), thresholds ordered by criticality
- [ ] Retries: 3 attempts max, below 10% per client, counter in metadata, one layer up only, "don't retry" error
- [ ] Degraded-response path; batch proxy; connection and health-check tuning
- [ ] Task survives 2x to 10x provisioned load

## Verbatim quotes

- "A moving target makes a poor metric for designing and implementing load balancing."
- "the backend can become overloaded even though the vast majority of its CPU is spent just rejecting requests!"
- "requests should only be retried at the layer immediately above the layer that is rejecting them."
- "A well-behaved backend, supported by robust load balancing policies, should accept only the requests that it can process and reject the rest gracefully."
