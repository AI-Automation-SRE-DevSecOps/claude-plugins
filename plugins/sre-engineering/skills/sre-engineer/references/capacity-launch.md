# Capacity Planning, Launches and Production Readiness

Sources: SRE Book ch.1 "Introduction", ch.2 "The Production Environment at Google", ch.18 "Software Engineering in SRE" (Auxon), ch.22, ch.27 "Reliable Product Launches at Scale", ch.32 "The Evolving SRE Engagement Model", App. B, App. E "Launch Coordination Checklist". SRE Workbook ch.11 "Managing Load", ch.12 "Introducing Non-Abstract Large System Design", ch.18 "SRE Engagement Model".

## Capacity planning

**Required steps** (Book ch.1):
1. An accurate **organic demand forecast** that extends past the lead time for acquiring capacity.
2. **Inorganic demand** from launches, marketing and events included in the forecast.
3. Regular **load testing** that maps raw capacity (CPU, servers) to service capacity.

Capacity is critical to availability, so the SRE team should own capacity planning and provisioning.

**N+2** (Book ch.2, App. B): serve peak traffic, possibly degraded, with the **two largest instances unavailable** at once, one for a planned update and one for an unplanned failure.

- Example (Book ch.2): about 100 QPS per task and about 3,470 QPS at peak need 35 tasks, so plan **37**. Split by region: USA 17, Europe 16, Asia 6. South America used N+1 to save 20%, relying on failover to other regions.
- Example (Book ch.22): a breaking point of 5,000 QPS per cluster and a 19,000 QPS peak need about **6 clusters** at N+2. Don't assume load spreads evenly by geography.
- N+2 assumes failures are independent. Shared failure domains (a top-of-rack switch, power, a control plane) break that assumption.

**Plan in resources, not QPS** (Book ch.21). Measure capacity as CPU (or the binding resource) at the latency SLO, for each datacenter. The launch checklist asks for capacity "per datacenter at max latency".

**Intent-based planning** (Book ch.18, Auxon): state the requirement, not the implementation. Moving up the levels:
1. "50 cores in clusters X, Y, Z"
2. "50 cores in any 3 clusters in region R"
3. "**Meet demand with N+2 per region**"
4. "Run at 5 nines"

Most of the benefit comes from level 3. Ask the planner for the intent and let it solve for placement.

**Launch spikes**: launch traffic has reached **15×** the estimate (Book ch.27). Pokémon GO load-tested at 5× its best estimate, and launch traffic was nearly 50× that estimate (Workbook ch.11). For launches, plan headroom, a ramp (invites, regions, percentages) and a load-shedding fallback.

**Simple forecasting model**

```python
import math

def tasks_needed(peak_qps: float, qps_per_task_at_slo: float, redundancy: int = 2) -> int:
    """N+2 by default: ceil(peak / capacity_per_task) + 2.
    qps_per_task_at_slo must come from a load test at the latency SLO, not from average CPU."""
    return math.ceil(peak_qps / qps_per_task_at_slo) + redundancy

def forecast_peak(current_peak: float, organic_growth_per_month: float, months: int,
                  inorganic_events: list[float] = ()) -> float:
    """Organic compound growth plus known launches or events, each as a multiplier on peak."""
    peak = current_peak * (1 + organic_growth_per_month) ** months
    for multiplier in inorganic_events:
        peak *= multiplier
    return peak

# Lead time to get capacity is 3 months, so forecast at least 3 months out, plus margin.
peak = forecast_peak(3_470, 0.04, months=4, inorganic_events=[1.2])   # a 20% marketing push
print(tasks_needed(peak, qps_per_task_at_slo=100))                     # per region, if regions are independent
```

Don't trigger scale-ups by fitting a line to CPU % and comparing it to 80%. Utilization depends on the request mix and hides queueing. Plan from **peak demand ÷ load-tested capacity per unit**, at N+2, per failure domain.

## Non-Abstract Large System Design (Workbook ch.12)

Reliability is "the most critical feature" of a production system. Design for it from the start. Iterate through these questions:
1. **Is it possible?** Get a design that works on one machine, ignoring scale.
2. **Can we do better?** Improve the algorithm or data structures.
3. **Is it feasible?** Scale it with real numbers: machines, disks, network, RAM.
4. **Is it resilient?** Survive the loss of a machine, a zone or a region within the SLO.
5. **Can we do better?** Revisit.

Rules: write down assumptions and **round aggressively**, and put concrete numbers on every component. Useful baselines: about 200 IOPS per HDD and about 25 ms per cross-datacenter Paxos operation, so at most about 40 sequential ops/s (Workbook ch.12). About 1 ms RTT within a datacenter, about 45 ms across the US and about 70 ms from New York to London (Book ch.23). The reasoning matters more than the exact figures.

## Launch Coordination Engineering (Book ch.27)

- A **launch** is "any new code that introduces an externally visible change to an application". Google ran up to 70 launches a week.
- **Launch Coordination Engineers (LCEs)** run a lightweight review. A low-risk launch, for example one with no new servers and under 10% traffic increase, gets a lighter process.
- **Every checklist question must be justified, "ideally by a previous launch disaster"**, and every action must be concrete. Review the checklist once or twice a year.

### Launch checklist (condensed from Book App. E and ch.27)

| Area | Questions |
|---|---|
| **Architecture** | Architecture diagram; request flow from client to backends; types of requests; **N+2** redundancy |
| **Machines and datacenters** | Which datacenters; machine and datacenter redundancy; failure domains |
| **Volume estimates** | Traffic and growth **6 months out**; launch spike; traffic mix; read/write ratio |
| **Capacity and performance** | Load test, end-to-end test, capacity **per datacenter at max latency**; plan for **10× growth**; growth alerts |
| **Load testing** | Load tests required for most launches, including the overload behavior |
| **System reliability and failover** | What happens on machine, rack, cluster and inter-datacenter network failure? For each backend: how to detect failure, restart without user impact, load balancing, rate limits, timeouts, retry and backoff |
| **Monitoring and server management** | Internal state monitoring, end-to-end behavior, alert management; **monitoring the monitoring**; no email alert storms |
| **Security** | Design review, code audit, spam/abuse risk, authentication, SSL |
| **Automation and manual tasks** | Methods to change servers, data and configs; release process; repeatable builds; **canaries under live traffic**; staged rollouts |
| **Growth and scaling** | Spare capacity; horizontal scaling; caches (latency or capacity?) |
| **External dependencies** | Third-party systems; partner throttling; behavior when a dependency fails; soft vs hard dependencies |
| **Schedule and rollout** | Hard deadlines; avoid **Mondays and Fridays**; marketing coordination; rollback plan |
| **Client behavior** | Exponential backoff with jitter; no synchronized polling; don't retry 4xx |
| **Feature flags** | Ramp from 1–10% of users; each change revertible on its own |

## Production Readiness Review (PRR) and engagement (Book ch.32, Workbook ch.18)

An SRE team takes on a service only after a PRR. **Simple PRR model** phases:
1. **Engagement**: agree scope, goals and SLOs.
2. **Analysis**: walk the launch checklist and best practices to find reliability gaps.
3. **Improvements and refactoring**: dev and SRE prioritize and fix the gaps.
4. **Training**: SREs learn the service through docs, sessions and Wheel of Misfortune.
5. **Onboarding**: gradually hand over operational responsibility.
6. **Continuous improvement**: keep tracking reliability; SRE can hand the service back.

Other models:
- **Early engagement**: SRE joins at design time. Dark-launch the service before it takes production traffic.
- **Frameworks and SRE platform**: SRE runs a platform that handles load shedding, monitoring and rollout. Developers are on call for their own code bugs. Staffing grows with the platform, not with the number of services.

**Ground rules for keeping SRE support** (Workbook ch.18):
- A hard limit on operational work
- SLOs agreed and measured
- A quarterly error-budget review
- Developers involved in daily operations, for example one developer in the on-call rotation after GA

When ground rules slip, escalate in steps: named engineers fix it, then a reliability hackathon, then a feature freeze, then "all hands on deck". Handing the service back to its developers is a valid outcome, and "disengaging from a service that doesn't need SRE counts as a positive outcome". (Book ch.32)

### PRR output template

```markdown
# PRR: <service>  ·  Date · SRE reviewers · Dev owners
## Scope and SLOs (link to the SLO document)
## Architecture and dependencies (diagram; hard vs soft dependencies; failure domains)
## Findings
| # | Area (checklist row) | Gap | Risk (budget impact) | Fix | Owner | Must fix before handoff? |
## Capacity: peak forecast, load-test result per unit, N+2 plan per region
## Monitoring and alerting: SLO burn alerts, playbooks, dashboards
## Release: canary, rollback, config safety
## Operations: on-call plan, toil estimate (< 50%), training plan
## Decision: accept / accept with conditions / not yet
```
