# SRE Workbook, Chapter 11: Managing Load

Source: https://sre.google/workbook/managing-load/
Authors: Cooper Bethea, Gráinne Sheerin, Jennifer Mace, Ruth King, with Gary Luo and Gary O'Connor. Prerequisite reading: SRE Book Ch. 19 (Load Balancing at the Frontend) and Ch. 20 (Load Balancing in the Datacenter).

## Core principles

- No single tool equalizes load. Google combines **load balancing, load shedding, and autoscaling**. All three serve "the same goal: to equalize and stabilize the system load". They are usually configured separately, but they are **not independent** (Combining Strategies to Manage Load, Fig. 11-8).
- **Run GCLB without DNS load balancing** (Google Cloud Load Balancing). DNS LB depends on clients expiring and refetching records, so GCLB uses **anycast** with a single VIP instead. Because there is only one VIP, DNS TTLs can be longer, which lowers latency further.
- **Measure load as close to the client as possible** (Pokémon GO, Future-proofing). Refused connections never appeared in inbound-request monitoring.
- Scaling up matters more and carries less risk than scaling down (Configuring Conservatively).
- Traffic management works best from "a holistic view of the interactions between systems" (Conclusion).

## Architecture components (GCLB stack)

- **Anycast / BGP**: Google announces IPs via BGP from many points, and packets go to the topologically nearest frontend. Two problems remain: nearby users can overwhelm a site, and BGP route recalculation ("flaps") can reset TCP connections.
- **Stabilized anycast**: each Maglev maps client IPs to the closest frontend site. If a packet lands at the wrong site, Maglev forwards it to a Maglev at the closest site.
- **Maglev**: a distributed packet-level load balancer that runs on commodity hardware. It spreads packets across the Maglev pool with ECMP, which gives **N + 1** redundancy instead of the **1 + 1** of traditional active/passive pairs. It computes a **5-tuple hash** (src address, dst address, src port, dst port, transport protocol) and checks its connection tracking table. It reuses the entry if the backend is still healthy, and otherwise falls back to **consistent hashing**. Maglev machines share no connection state, and it scales "by packet rather than by number of connections".
- **GSLB**: balances connections to GFEs and requests to backends across clusters. It knows backend health and drains traffic away from failed clusters.
- **GFE**: terminates TCP/SSL, routes on the HTTP header and URL path, re-encrypts traffic to backends, and health-checks them. A NACK or a health-check timeout stops traffic to that backend. It keeps persistent sessions to recently active backends.
- **Lame duck mode**: a backend fails its health checks while it finishes in-flight requests. This drains it gracefully.
- **Low latency**: HTTPS needs **two network round trips**, so Maglev and GFE sit at the network edge and terminate SSL close to the user.
- **High availability**: GCLB's availability **SLA is 99.99%**. Canarying and gradual rollouts let the load balancer route around canaries that crash or fail health checks. A canary with a nonfatal regression can be removed from the load balancer by an administrator.

## Concrete numbers (as written)

| Item | Value |
|---|---|
| Google infra build-out | "past 18 years" |
| GCLB availability SLA | 99.99% |
| Maglev redundancy | N + 1 (vs. 1 + 1 active/passive) |
| HTTPS handshake | two network round trips |
| Pokémon GO pre-launch load test | up to **5x** most optimistic estimate |
| Actual launch RPS | nearly **50x** that estimate |
| True client demand found during migration | **200% higher** than previously observed |
| GCLB worldwide capacity lost to GFE regression | effectively reduced by **50%** |
| Synchronized retry spikes | **20×** previous global RPS peak |
| Pokémon GO size after GCLB migration | single largest GCLB service within two days |
| Dressy baseline | 3 regions at 90 RPS each |
| Dressy at 11:00 | region A at 120 RPS |
| Dressy at 11:10 | A at 400 RPS; B and C dipped to 40 RPS |
| Dressy shedding | 10% → 20% → 50% of requests rejected, CPU constant |
| Dressy per-request CPU (as seen by the LB) | 10 times lower in region A |
| Dressy shedding cap | A serving 240 RPS at **80% CPU**; B/C 120 RPS |
| Proposed fix | count each error as **120% CPU** ("any number over 100 will work") |
| Social-media surge example | five-fold traffic increase |

## Case Study 1: Pokémon GO on GCLB

- Before the migration, Pokémon GO ran a regional NLB in front of a Kubernetes cluster with an Nginx L7 proxy tier. That setup had three problems: SSL terminated at Nginx (two round trips), slow clients exhausted proxy buffers, and a packet-level proxy could not mitigate SYN floods.
- A large SYN flood made the migration urgent. Cutover happened in a traffic trough. As traffic ramped to peak, the outage unfolded in this order:
  1. Backends became slow instead of refusing requests.
  2. The load balancer retried GET requests.
  3. GFE SSL reconnection logic regressed.
- Client retry policy at the time: "a single immediate retry, followed by constant backoff". Fast errors synchronized clients into a **thundering herd**.
- Mitigations:
  - Isolate the GFEs serving Pokémon GO from the main pool.
  - Enlarge the isolated pool.
  - Add administrative overrides that rate-limit the traffic the load balancers accept for the service.
- Fixes afterward:
  - Niantic added **jitter and truncated exponential backoff**.
  - Google now treats GFE backends as a significant source of load and load-tests for GFE degradation caused by slow backends.

## Autoscaling practices

- **Handling Unhealthy Machines**: autoscalers average utilization over all instances, including unhealthy ones (slow warm-up, zombies), so scaling may never trigger. Fixes:
  - Autoscale on the **capacity metric observed by the load balancer**.
  - Use a **cool-down period** (GCE term) so new instances are counted only once they are healthy.
  - Pair autoscaling with **autohealing**, and give restarted instances enough time to become healthy.
  - Remember that "Creating new instances is never instant."
- **Working with Stateful Systems**: horizontal scaling does not help an overburdened session path. Use task-level routing such as consistent hashing. Vertical autoscaling can absorb short-term hotspots, but use it with caution: it is usually uniform across instances, so low-traffic instances may grow unnecessarily large.
- **Configuring Conservatively**: autoscalers react quickly to increases and are deliberately slow to scale down. Keep the service far from bottlenecks such as CPU. User-facing services should reserve spare capacity for both overload protection and redundancy.
- **Setting Constraints**: set **minimum and maximum bounds**, and make sure quota covers those limits.
- **Including Kill Switches and Manual Overrides**: the kill switch should be "easy, obvious, fast, and well documented", and on-call engineers must know how to scale manually.
- **Avoiding Overloading Backends**: do a dependency analysis before deploying an autoscaler and use the results to set its limits. Microservices that share quota can starve each other, so cap scaling ahead of time or give each microservice its own quota (separate projects).
- **Avoiding Traffic Imbalance**: regional MIG autoscalers rebalance zone sizes across a region, which evens out per-zone quota and diversifies failure domains.

## Anti-patterns Google warns against

- Relying on DNS load balancing that needs client cooperation.
- Load-testing only to optimistic estimates (5x was not enough).
- Backends that go slow instead of failing fast, combined with load-balancer retries.
- Client retries that fire immediately, use constant backoff, or have no jitter.
- Autoscaling on CPU with no upper bound: a CPU-burning bug or stuck requests on a failing dependency can scale until **all quota is wasted**, and the extra load can stop the dependency from recovering.
- Enabling load shedding and utilization-aware load balancing separately, so errors look like "efficient" cheap requests (Dressy).
- Load balancing plus autoscaling with no per-location minimum. This creates a **positive feedback loop** that concentrates all capacity in one location.
- Shedding load before the autoscaler has had a chance to act.
- RPCs with no deadline. Default deadlines are "a very large number", so resources are held, latency rises, and memory can run out.

## Checklist: precautions when combining tools (Lessons Learned)

- [ ] Examine how each new load-management tool interacts with the existing ones, and instrument where they intersect.
- [ ] Add monitoring that detects feedback loops.
- [ ] Coordinate emergency shutdown triggers across load-management systems, and consider automatic shutdown triggers.
- [ ] Make the load balancer error-aware: penalize errors above 100% utilization.
- [ ] Set a minimum number of instances per location to keep spare capacity for failover.
- [ ] Set thresholds so the system **autoscales before load shedding kicks in**.
- [ ] Set RPC deadlines. Servers should terminate requests that take too long, and clients should cancel requests they no longer need. Suggest a default deadline in a comment in the `.proto` file.
- [ ] Plan mitigations ahead: flags, changed defaults, expensive logging, and exposed traffic-management parameter values.

## Short verbatim quotes

- "No service is 100% available 100% of the time"
- "Creating new instances is never instant."
- "Your autoscaling kill switch functionality should be easy, obvious, fast, and well documented."
- "the load balancer didn't know that the 'efficient' requests were errors because the load shedding and load balancing systems weren't communicating."
- "no amount of load shedding, autoscaling, or throttling will save our services when they all fail in sync."
- "In Google's experience, there are no perfect traffic management configurations."
