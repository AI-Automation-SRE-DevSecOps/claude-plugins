# SRE Book, Chapter 20: Load Balancing in the Datacenter

Source: https://sre.google/sre-book/load-balancing-datacenter/ (Alejandro Forero Cuervo; ed. Sarah Chavis)

Scope: how *client tasks* pick which *backend task* serves each query **inside one datacenter**. Switches/packet routing and datacenter selection are out of scope (datacenter selection is Ch. 19). Clients do the balancing themselves: they hold long-lived connections and decide per request.

## Core principles

- **Waste comes from imbalance (The Ideal Case).** You can only send traffic to a datacenter until the *most loaded* task hits its limit. Idle capacity on every other task is "reserved, but unused."
- **Find and avoid unhealthy tasks first (Identifying Bad Tasks).** Clients see a backend as one of three states: *Healthy*, *Refusing connections*, or *Lame duck*. Lame duck means the task is listening and can serve, but is asking clients to stop sending requests.
- **Subsetting (Limiting the Connections Pool).** Each connection costs memory and CPU at both ends because of health checks. Subsetting limits how many backends each client connects to. A good subset algorithm spreads backends evenly across clients, keeps churn low when tasks restart or fail, and copes with resizes without knowing client or backend counts in advance.
- **Policies decide with partial, stale information (Load Balancing Policies).** Clients only see their own requests. The more real backend state a policy uses, the better it does.

## Concrete numbers, thresholds, formulas

| Item | Value as written |
|---|---|
| Smallest services | at least 3 processes (fewer means "losing 50% or more of your capacity if you lose a single machine") |
| Typical service | 100 to 1,000 processes; largest more than 10,000 |
| Wasted CPU | sum over all tasks i of (CPU[0] – CPU[i]), where task 0 is the most loaded |
| Waste example | reserve 1,000 CPUs, able to use only about 700 |
| Flow-control active-request limit | "For most backends, 100 is a reasonable limit" (per client, per backend connection) |
| Lame duck propagation | "typically in 1 or 2 RTT," including to inactive clients, which still send UDP health checks |
| Lame duck shutdown interval | rule of thumb 10s to 150s, depending on client complexity |
| Typical subset size | 20 to 100 backend tasks |
| Subsetting imbalance cost | if subsetting overloads one backend by 10%, all backends must be overprovisioned by 10% |
| Random subsetting, 300 clients/300 backends, 30% subset (90) | least loaded 63% of average (57 connections), most loaded 121% (109) |
| Random subsetting, 10% subset (30) | least 50% (15 connections), most 150% (45) |
| Random subsetting for an even spread | would need subsets "as large as 75%," which is impractical |
| Deterministic subsetting | backend client counts differ "by at most 1" in most cases; with 300 clients, 10 of 300 backends, every backend gets exactly the same number of connections |
| Simple Round Robin spread | "up to 2x" CPU, least to most loaded |
| Query cost range | most expensive requests take 1000x (or more) CPU than the cheapest; interfaces often allow 100, 1,000, or 10,000x |
| Java backend example | around 15 ms CPU on average, but some queries up to 10 seconds |
| Machine equivalence example | 2 CPU units on slow machine X = 0.8 CPU units on fast machine Y, which led to GCU (Google Compute Units) |
| Antagonistic neighbors | performance differences "up to 20%" |
| Task restarts | need much more resources "for a few minutes" |
| Least-Loaded RR at scale | most loaded task uses about 2x the CPU of the least loaded, "about as poorly as Round Robin" |

**Deterministic subsetting algorithm (verbatim code in the chapter):**
```
subset_count = len(backends) / subset_size
round = client_id / subset_count
random.seed(round); random.shuffle(backends)
subset_id = client_id % subset_count
start = subset_id * subset_size
return backends[start:start + subset_size]
```
Clients are grouped into rounds of `subset_count` consecutive clients. Within a round, each backend goes to exactly one client. The last round may not have enough clients. Worked example: 12 backends with subset size 3 gives 4 subsets per round. With 10 clients, every backend ends up with 2 or 3 clients.

## Recommended practices and procedures

- **Clean shutdown with lame duck (A Robust Approach):**
  1. The job scheduler sends SIGTERM.
  2. The task enters lame duck through an RPC API call made in the SIGTERM handler.
  3. Requests already in flight, including ones that arrive before clients notice the state, finish normally.
  4. The active-request count drains to zero.
  5. After the configured interval (10–150s), the task exits cleanly or the scheduler kills it.
- **Lame duck at startup:** let clients connect during long initialization, then signal readiness explicitly. Also keep restarted servers in lame duck and prewarm them, for example to trigger JIT on Java, until performance is normal.
- **Use a larger subset** when there are far fewer clients than backends, or when clients send bursty or fan-out-heavy traffic (Picking the Right Subset).
- **Shuffle, with a different seed per round.** If you don't shuffle, a rolling update in task order can take a client's whole subset down at once. If every round uses the same seed, a failed backend's load lands only on the rest of its subset (`subset_size - N`), not on all backends.
- **Cap per-request work** in the service interface when query costs vary widely, for example by paginating ("most recent 100 emails"). Paginate over a fixed view of the data so results stay consistent.
- **Normalize heterogeneous hardware** by scaling CPU reservations by machine type (GCU).
- **Prefer Weighted Round Robin.** Backends report query rate, error rate, and utilization (usually CPU) in every response, health checks included. Clients periodically update a "capability" score per backend, and failed requests count as a penalty.

## Anti-patterns Google warns against

- **Relying only on the active-request limit.** It catches only extreme overload. It has also "backfired, causing all backend tasks to become unreachable," and it can't tell an unhealthy backend from a slow one.
- **Opening a connection per request.** It has significant resource and latency costs.
- **Connecting every client to every backend.** Health-check overhead wastes resources for little gain.
- **Random subsetting.** Load spreads very unevenly at practical subset sizes.
- **Simple Round Robin at scale.** Causes: small subsetting, varying query costs, machine diversity, and unpredictable performance (antagonistic neighbors, task restarts).
- **Least-Loaded RR without counting errors.** It causes "sinkholing": an unhealthy task that fails requests quickly looks lightly loaded and attracts traffic. Fix: count recent errors as active requests. Two other limits remain: an active-request count is a poor proxy for capacity when requests mostly wait on I/O, and each client sees only its own requests.

## Checklist (derived from the chapter)

- [ ] Backends support lame duck, entered from the SIGTERM handler
- [ ] Drain interval set to cover typical requests (10–150s)
- [ ] Inactive connections still send UDP health checks
- [ ] Deterministic subsetting with a per-round shuffle seed; subset of 20–100
- [ ] Per-request cost bounded (pagination) where costs vary by orders of magnitude
- [ ] Policy uses backend-reported utilization and errors (Weighted RR); errors are penalized

## Short verbatim quotes

- "In this case 'wasted' means reserved, but unused."
- "if subsetting overloads one backend by 10%, the whole set of backends needs to be overprovisioned by 10%."
- "Round Robin can result in a spread of up to 2x in CPU consumption from the least to the most loaded task."
- "We say that the unhealthy task is now sinkholing traffic."
- "If your load balancing policy can't adapt to unforeseen performance limitations, you will inherently end up with a suboptimal load distribution"
