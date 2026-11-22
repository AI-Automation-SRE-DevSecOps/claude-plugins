# SRE Book Ch. 19 — Load Balancing at the Frontend

Source: https://sre.google/sre-book/load-balancing-frontend/ (written by Piotr Lewandowski, edited by Sarah Chavis). Scope: load balancing user traffic *between* datacenters. Ch. 20 covers load balancing *inside* a datacenter. The examples assume HTTP over TCP. The chapter says most of the mechanisms also apply to stateless services such as DNS over UDP.

## Core principles

- **No single point of failure, even with infinite power** (§ "Power Isn't the Answer"). Even an ideal machine is still limited by physics: the speed of light in fiber puts an upper bound on how fast data can be served over distance.
- **"Optimal" depends on context.** It depends on three things: the hierarchical level (global vs. local), the technical level (hardware vs. software), and the nature of the traffic.
- **Traffic type decides the objective.** For search, *latency* matters most, so the request goes to the nearest available datacenter measured by RTT. For video upload, *throughput* matters most, so it may take a different path, for example an underutilized link, "at the expense of latency".
- **Global vs. local.** Inside a datacenter, all machines are assumed to be equally distant from the user, so the goal becomes resource utilization and keeping any single server from overloading.
- **Other factors.** Traffic may go to a slightly farther datacenter to keep caches warm. Non-interactive traffic may go to a completely different region to avoid congestion.
- **Balance at multiple levels.** First DNS, then the VIP / network load balancer layer.

## Load Balancing Using DNS

- The simplest form is returning multiple A/AAAA records and letting the client pick. This gives little control: records are picked randomly, and each gets roughly equal traffic. SRV records support weights and priorities, but they "have not yet been adopted for HTTP."
- Clients usually can't tell which address is closest. Mitigations: an **anycast** address for authoritative nameservers, and then a **map of networks to physical locations**. The cost is a more complex DNS server plus a pipeline to keep the map up to date.
- **The recursive resolver in the middle has three implications:**
  1. *Recursive resolution of IP addresses.* The authoritative server sees the resolver's IP, not the user's. The fix is **EDNS0 client-subnet** [Con15]. It was not yet an official standard at the time, but OpenDNS and Google already supported it. A single resolver may serve anything from one office to a continent. Example: a national ISP whose nameservers sit in one datacenter gets answers tuned to that datacenter.
  2. *Nondeterministic reply paths.*
  3. *Additional caching complications.* One reply may reach one user or thousands. Google handles this in two ways: (a) it keeps a continuously updated list of known resolvers with the approximate size of the user base behind each, and (b) it estimates the geographic distribution of the users behind each tracked resolver. When users are spread widely, it optimizes for the majority.
- **"Best location" is more than the nearest location.** The chosen datacenter must have enough capacity for the users likely to receive that reply, and it must be healthy, with no power or network problems. To do this, the authoritative DNS is integrated with global control systems that track traffic, capacity, and infrastructure state.
- **TTL.** Authoritative servers can't flush resolver caches, so records need a "relatively low TTL". The TTL sets a lower bound on how fast changes reach users. Footnote 104: not all resolvers respect TTL.
- **DNS alone is not sufficient**, though it is still "the simplest and most effective way to balance load before the user's connection even starts."

## Concrete numbers / formulas

| Item | Value (as written) |
|---|---|
| DNS reply size limit | **512 bytes**, per RFC 1035 [Moc87]. This caps how many addresses fit in one reply, "almost certainly less than our number of servers." Footnote 105: otherwise users must open a TCP connection just to get the IP list. |
| Naive connection-to-backend mapping | `id(packet) mod N`, where `id` derives a connection ID from the packet and N is the number of configured backends |
| Failure of naive hashing | When N becomes N-1, the mapping changes to `id(packet) mod N-1`, so "Almost every packet suddenly maps to a different backend". Without shared state, almost all connections reset. |
| Consistent hashing | Proposed in **1997** [Kar97] |
| GRE encapsulation overhead | **24 bytes** for IPv4+GRE |

## Load Balancing at the Virtual IP Address

- A **VIP** isn't assigned to any one interface. It is shared across devices but looks like a single IP to the user. This hides the backend count and makes maintenance (upgrades, adding machines) invisible to the user.
- The key component is the **network load balancer**, which forwards packets to a backend behind the VIP.
- **Backend selection:**
  - *Least-loaded* is intuitive, but it breaks for stateful protocols. The balancer would have to track every connection.
  - *Stateless hash `mod N`* breaks when the backend set changes (see table).
  - ***Consistent hashing*** keeps the mapping relatively stable as backends are added or removed. **Google's practice:** "usually use simple connection tracking, but fall back to consistent hashing when the system is under pressure (e.g., during an ongoing denial of service attack)."
- **Forwarding methods:**
  - *NAT* needs a tracking-table entry for every connection, which "precludes having a completely stateless fallback mechanism."
  - *Layer-2 rewrite / Direct Server Response (DSR)* changes only the destination MAC. The backend sees the original IPs and replies directly to the client. This saves a great deal when requests are small and replies large (most HTTP), and it keeps no state on the LB. The catch is that every LB and backend must share one broadcast domain. Google "outgrew this solution quite some time ago."
  - ***GRE packet encapsulation*** is Google's current approach (Maglev, [Eis16]; [Han94]). The LB wraps the packet in IP+GRE addressed to the backend, and the backend strips the outer layer. No shared broadcast domain is needed, so the LB and backend can be on different continents as long as a route exists.
- **Cost of encapsulation:** a 24-byte overhead that can push packets over the MTU and force fragmentation. Inside the datacenter this can be avoided with a larger MTU, which requires a network that supports large PDUs.

## Recommended practices (checklist)

- [ ] Put DNS LB first (geo-aware via anycast, a network-location map, and EDNS0 client-subnet), then VIP/network LB.
- [ ] Choose the routing objective per traffic class: latency (nearest by RTT) vs. throughput (underutilized links).
- [ ] Feed live capacity and health signals into the authoritative DNS's datacenter choice.
- [ ] Track resolvers: the user population behind each and its geographic spread.
- [ ] Keep DNS TTLs relatively low, and expect some resolvers to ignore them.
- [ ] Keep DNS replies within 512 bytes.
- [ ] Use connection tracking normally, with consistent hashing as a stateless fallback under pressure or DDoS.
- [ ] Prefer encapsulation (GRE) over NAT or L2 DSR at scale, and plan the MTU or jumbo frames to absorb the overhead.

## Anti-patterns Google warns against

- Relying on a single point of failure ("putting all your eggs in one basket").
- Relying on plain multi-A-record round robin, or on DNS alone.
- Optimizing DNS answers for the resolver's location instead of the user's.
- Sending users to the closest datacenter without checking its capacity and health.
- Hashing with `mod N`, which remaps almost every connection when the backend set changes.
- NAT-based forwarding, if you need a fully stateless fallback.
- L2/DSR at a scale that outgrows a single broadcast domain.

## Short verbatim quotes

- "when you're dealing with large-scale systems, putting all your eggs in one basket is a recipe for disaster."
- "DNS is still the simplest and most effective way to balance load before the user's connection even starts."
- "it should be clear that load balancing with DNS on its own is not sufficient."
- "Almost every packet suddenly maps to a different backend!"
- "load balance early and load balance often—but the difficulty is in the details"
