# Chapter 23: Managing Critical State: Distributed Consensus for Reliability

Source: https://sre.google/sre-book/managing-critical-state/ (SRE Book, by Laura Nolan, edited by Tim Harvey)

## Core principles

- **Use proven consensus for coordination problems.** Leader election, group membership, distributed locking/leasing, reliable queuing/messaging, and any critical shared state are all forms of distributed consensus. Solve them "only using distributed consensus algorithms that have been proven formally correct, and whose implementations have been tested extensively" (Case Study 3).
- **CAP:** a system cannot have consistency, availability, and partition tolerance all at once. Partitions are inevitable, so the real question is how consistency and availability trade off for your application (CAP Theorem).
- **Correctness comes first.** "System designers cannot sacrifice correctness in order to achieve reliability or performance, particularly around critical state."
- **BASE stores** use multimaster replication with resolution like "latest timestamp wins", which can misbehave under clock drift or partitions.
- **The model that matters is asynchronous, crash-recover consensus.** Asynchronous means message delays can be unbounded. Crash-recover is "much more useful" than crash-fail because most real failures are transient. Byzantine failures cost more to handle and happen less often (How Distributed Consensus Works).
- **FLP impossibility:** no asynchronous consensus algorithm can guarantee progress over an unreliable network in bounded time. In practice you get bounded time from enough healthy replicas and connectivity plus randomized backoff. "The protocols guarantee safety, and adequate redundancy in the system encourages liveness."
- **Consensus is a low-level primitive.** On its own it lets nodes agree on a value once. Build on higher-level components (RSMs, datastores, config stores, queues, lock and leader-election services) and prefer consensus *as a service* (Chubby, Zookeeper, Consul, etcd) over libraries (System Architecture Patterns).
- **Replicated state machine (RSM):** runs the same operations in the same order on several processes, with the order set by consensus. Any deterministic program can be made highly available this way.

## Concrete numbers, thresholds, formulas

| Item | Value as written |
|---|---|
| Majority quorum | **2f + 1** replicas tolerate **f** failures |
| Byzantine fault tolerance | **3f + 1** replicas tolerate **f** failures |
| Minimum replicas (non-Byzantine) | **3**. With 2 there is no failure tolerance. 3 tolerate 1 failure |
| Recommended | **5** replicas, which tolerate up to 2 failures |
| Replacement trigger | 4 of 5 remaining needs no intervention. At 3 left, "an additional replica or two should be added" |
| 5 vs 6 replicas | 5: quorum is 3, so 2 (**40%**) can be down. 6: quorum is 4, so only **33%** can be down. Adding a replica can *reduce* availability |
| Typical Zookeeper/Chubby deployment | 5 replicas |
| RTT within a datacenter | on the order of **1 ms** |
| Typical RTT within the US | **45 ms** |
| RTT New York to London | **70 ms** |
| Initial TCP/IP window | **4 to 15 KB** |
| Disk log write | between **one and several ms** |
| Disk-bound throughput | ~**10 ms** small random write gives ~**100 consensus ops/sec** (assuming negligible RTT and proposer logging in parallel) |
| EU to central-US RTT | about **50% higher** than EU to east-coast RTT |
| Hierarchical quorum example | **9 replicas in 3 groups of 3**. Quorum = majority of groups. A group counts if a majority of its members are up |

**Latency of one Multi-Paxos operation** (Disk Access): one disk write on the proposer, then parallel messages to the acceptors, then parallel disk writes at the acceptors, then the return messages. Disk writes must happen before an acceptor sends *Accepted* and before the proposer sends *Accept*, because Accept is an implicit Accepted.

**Multi-Paxos:** with a stable leader, consensus takes one round trip from the proposer to a quorum.

## Recommended practices and procedures

- **Paxos:** proposers need unique sequence numbers, and acceptors must journal to persistent storage on each acceptance.
- **Leader election tuning:** balance the unavailability you get with no leader against the risk of dueling proposers. "Introducing randomness is the best approach". Raft is cited as a good model.
- **Leases over locks:** use "renewable leases with timeouts instead of indefinite locks". Queues should use a lease system rather than removing a task outright when it is claimed. Most applications should use distributed transactions instead of raw locks.
- **Implement queues as RSMs** so that losing the queue doesn't halt the whole system.
- **Scaling reads:** a strongly consistent read requires one of these: (1) a read-only consensus operation, (2) a read from a replica guaranteed to be up to date, such as a stable leader, or (3) **quorum leases**. A quorum lease grants a read lease to a quorum of replicas, and writes must be acknowledged by every replica in that read quorum. Stale reads from any replica are acceptable when staleness only costs extra work (Photon).
- **Performance techniques:** batching, pipelining, a single combined consensus/RSM log, and regional proxies that hold persistent TCP connections.
- **Placement:** choose replica count by weighing reliability, how often planned maintenance happens, risk, performance, and cost. Choose locations by weighing failure domains against latency. Client-perceived latency is the most important performance measure.
- **Balance leaders** for different shards evenly across datacenters, because the leader's outgoing bandwidth is the bottleneck. Check that inter-datacenter links can absorb a mass leader failover.
- **Back up snapshots elsewhere**, even for consensus systems spread across many failure domains. Two failure domains can never be escaped: the software itself and human error.

## Anti-patterns Google warns against

- **Heartbeats/timeouts plus STONITH for leader election** (Case Study 1). STONITH is "conceptually unsound": when the network is slow or lossy, you can end up with both nodes active (corruption) or both dead (unavailability).
- **Escalating failover to humans** (Case Study 2). It hurts availability, scales poorly, and pages people when they are already overloaded. If consensus can't elect a master, a human isn't better placed to.
- **Gossip-based group membership** (Case Study 3). Each side of a partition elects its own master.
- **Relying on timestamps** for data freshness, because clocks can't be guaranteed to agree.
- **Assuming Fast Paxos is always faster.** With a high client-to-acceptor RTT it is *slower* than Classic Paxos.
- **Adding a 6th replica, especially in a datacenter that already has one.**
- **Routing reads to the nearest replica only**, which risks cascading overload.
- **Unintentionally co-locating leaders** through heuristics such as placing them near clients, on the fastest machines, or on the longest-running processes.
- **Majority quorums with one "linchpin" replica** where quorums overlap. Losing it sharply increases latency, and hierarchical quorums are the fix.
- **Forced reconfiguration after quorum loss.** It is possible, but "the possibility of data loss always remains."

## Monitoring checklist (Monitoring Distributed Consensus Systems)

1. Number of members running in each group, and each one's health. A process can be running yet make no progress.
2. Persistently lagging replicas. A healthy member can be recovering, lagging, up to date, or the leader.
3. Whether a leader exists. With no leader, a Multi-Paxos system is "totally unavailable."
4. Number of leader changes, based on term/view number. Rapid increases mean flapping. A *decrease* could signal a serious bug.
5. Consensus transaction number, which should increase over time.
6. Number of proposals seen and number agreed upon.
7. Throughput and latency.

For performance work, also track proposal-acceptance latency, inter-site network latency, acceptor durable-logging time, and bytes accepted per second.

## Short verbatim quotes

- "Leader election is a reformulation of the distributed asynchronous consensus problem, which cannot be solved correctly by using heartbeats."
- "Ad hoc means of solving these sorts of problems (such as heartbeats and gossip protocols) will always have reliability problems in practice."
- "Most system downtime is a result of planned maintenance"
- "There are two failure domains that you can never escape: the software itself, and human error"
- "any lesser approach is a ticking bomb waiting to explode in your systems."
