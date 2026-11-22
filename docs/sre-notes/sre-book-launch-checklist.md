# Appendix E — Launch Coordination Checklist (Google SRE Book)

Source: https://sre.google/sre-book/launch-checklist/ (SRE Book, 2017, CC BY-NC-ND 4.0)

**What this page is:** a short appendix. It contains only an intro sentence and a bulleted checklist with ten headings. It has no prose rationale, formulas, or tables. The explanation of *how* the checklist is used (Launch Coordination Engineers, the launch process) is in Chapter 27, "Reliable Product Launches at Scale," not here. Anything below that goes beyond the literal checklist is labeled as interpretation.

The page itself says: "This is Google's original Launch Coordination Checklist, circa 2005, slightly abridged for brevity."

## Core principles (inferred from the checklist's structure)

The page never states principles. These follow from what it chooses to list:
- **Think about failure before launch.** A whole section ("System reliability and failover") is "What happens when" questions about machines, racks, clusters, and networks failing.
- **Plan capacity for growth, not just for the launch.** Estimates go "6 months out," and the growth section calls for "10x growth."
- **A launch affects other systems.** Several items cover impact on "other services we care most about," third parties, syndicated partners, and mail systems.
- **Use a controlled rollout.** The checklist names "canaries under live traffic" and "staged rollouts."
- **Monitoring is itself a dependency.** It lists "Monitoring the monitoring."

## Concrete numbers / thresholds (exact, as written)

| Item | Exact wording | Section |
|---|---|---|
| Redundancy | "N+2 redundancy" | Machines and datacenters |
| Forecast horizon | "launch 'spike,' traffic mix, 6 months out" | Volume estimates, capacity, and performance |
| Capacity test condition | "capacity per datacenter at max latency" | Volume estimates, capacity, and performance |
| Growth headroom | "Spare capacity, 10x growth, growth alerts" | Growth issues |
| Checklist vintage | "circa 2005, slightly abridged" | Intro |

Notes on accuracy:
- The figure is **N+2**, not N+1.
- **"10x growth"** is a checklist prompt to consider. The page does not say how much capacity to provision.
- The page defines no SLO, availability, or latency targets.

## Recommended practices & procedures (by section heading)

**Architecture:** draw an architecture sketch covering types of servers and types of client requests. Include programmatic client requests.

**Machines and datacenters:** machines and bandwidth, datacenters, N+2 redundancy, network QoS. New domain names and DNS load balancing.

**Volume estimates, capacity, and performance:** HTTP traffic and bandwidth estimates, including the launch spike and traffic mix, 6 months out. Load test, end-to-end test, and capacity per datacenter at max latency. Impact on the other services you care most about. Storage capacity.

**System reliability and failover:**
- What happens when a machine dies, a rack fails, a cluster goes offline, or the network fails between two datacenters.
- For each type of server that talks to backends:
  - How to detect that backends have died, and what to do then.
  - How to terminate or restart without affecting clients or users.
  - Load balancing, rate-limiting, timeout, retry, and error-handling behavior.
- Data backup/restore and disaster recovery.

**Monitoring and server management:** monitor internal state and end-to-end behavior, and manage alerts. Monitor the monitoring. Cover financially important alerts and logs. Tips for running servers in a cluster environment.

**Security:** security design review, security code audit, spam risk, authentication, SSL. Prelaunch visibility/access control. Various types of blacklists.

**Automation and manual tasks:** methods and change control for updating servers, data, and configs. Release process, repeatable builds, canaries under live traffic, staged rollouts.

**Growth issues:** spare capacity, 10x growth, growth alerts. Scalability bottlenecks, linear scaling, scaling with hardware, changes needed. Caching and data sharding/resharding.

**External dependencies:** third-party systems, monitoring, networking, traffic volume, launch spikes. Graceful degradation, and how not to accidentally overrun third-party services. Playing nice with syndicated partners, mail systems, and services within Google.

**Schedule and rollout planning:** hard deadlines, external events, Mondays or Fridays. Standard operating procedures for this service and for other services.

## Anti-patterns Google warns against

- **Alert storms against mail servers:** "Don't crash mail servers by sending yourself email alerts in your own server code" (Monitoring and server management). This is the page's only explicit "don't."
- **Accidentally overrunning third-party services** (External dependencies).
- **Restarts that affect users:** the checklist asks how to terminate or restart "without affecting clients or users."
- **Launching on risky days:** "Mondays or Fridays" is listed as a scheduling concern. The page doesn't say why. A reasonable reading is that start and end of week carry rollout and staffing risk.
- **Unmonitored monitoring:** implied by "Monitoring the monitoring."

## Template / checklist (condensed, reusable)

```
[ ] Architecture: sketch, server types, request types, programmatic clients
[ ] Machines/DC: bandwidth, datacenters, N+2, network QoS, DNS/domains, DNS LB
[ ] Capacity: traffic + bandwidth est. (spike, mix, 6 months out); load test;
    e2e test; capacity/DC at max latency; impact on key services; storage
[ ] Failover: machine/rack/cluster/inter-DC network loss; per-backend detection
    + response; graceful restart; LB, rate-limit, timeout, retry, error handling;
    backup/restore; DR
[ ] Monitoring: internal + end-to-end; alert management; monitor the monitoring;
    financially important alerts/logs; no self-email alert storms
[ ] Security: design review, code audit, spam, authN, SSL, prelaunch ACLs, blacklists
[ ] Automation: change control (servers/data/configs), release process,
    repeatable builds, live-traffic canaries, staged rollouts
[ ] Growth: spare capacity, 10x, growth alerts, bottlenecks, linear scaling,
    caching, sharding/resharding
[ ] External deps: third parties, launch spikes, graceful degradation,
    don't overrun partners, mail systems, internal services
[ ] Schedule: hard deadlines, external events, Mondays/Fridays, SOPs
```

## Short verbatim quotes

- "This is Google's original Launch Coordination Checklist, circa 2005, slightly abridged for brevity"
- "Machines and bandwidth, datacenters, N+2 redundancy, network QoS"
- "Load test, end-to-end test, capacity per datacenter at max latency"
- "Spare capacity, 10x growth, growth alerts"
- "Don't crash mail servers by sending yourself email alerts in your own server code"
- "How to terminate or restart without affecting clients or users"
- "Hard deadlines, external events, Mondays or Fridays"
