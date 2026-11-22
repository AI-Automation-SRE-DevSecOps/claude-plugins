# Toil and Automation

Sources: SRE Book ch.5 "Eliminating Toil", ch.7 "The Evolution of Automation at Google", ch.13 (the Diskerase incident). SRE Workbook ch.1 "How SRE Relates to DevOps", ch.6 "Eliminating Toil".

## What toil is (Book ch.5)

Toil is work tied to running a production service that tends to be:

| Attribute | Meaning |
|---|---|
| **Manual** | A human runs it. Running a script by hand still counts; what matters is hands-on time, not elapsed time |
| **Repetitive** | Done again and again; not a novel problem |
| **Automatable** | A machine could do it. If it needs human judgment every time, it may not be toil, though frequent judgment calls signal a design problem |
| **Tactical** | Interrupt-driven and reactive rather than strategic |
| **No enduring value** | The service is in the same state afterwards |
| **O(n) with service growth** | Grows with traffic, users or the number of services |

Toil is a spectrum. The more of these traits a task has, the more toil it is. **Overhead is not toil**: meetings, planning, HR work, code review and grooming the bug queue are overhead.

Examples: manually restarting tasks, ticket-driven quota changes, hand-run releases, copying config between environments, and answering the same alert repeatedly.

## The 50% cap

- Keep **operational work (toil plus other ops) at or below 50%** of each SRE's time. Measure it as an average over a few quarters, not per week. (Book ch.5; Workbook ch.6)
- Google's surveys found **average toil around 33%**, ranging from 0% to 80%. The biggest sources were interrupts (non-urgent service messages), then on-call response, then releases and pushes.
- On-call alone sets a floor of about **2 / rotation size**: about 33% for a 6-person rotation, 25% for 8.
- The cap is a **guarantee of engineering time**, not just a limit. When ops work goes over it, the excess goes back to the dev team.
- **Growth target**: a well-designed service should handle **10× growth with no extra work** other than one-time resource additions. (Book ch.5)
- **Murphy-Beyer effect**: once everything automatable is automated, what's left is the hard, non-automatable work. Teams then take on more services, still within the 50% cap. (Workbook ch.1)

## Measuring toil (Workbook ch.6)

1. **Identify** toil from tickets, pages, on-call handoffs, surveys and time-tracking samples.
2. **Choose an objective unit**: minutes of human effort per occurrence, including the cost of context switching. "A 20-minute interruption" costs far more than 20 minutes of work. (Book ch.29)
3. **Track it** before, during and after a reduction project, and report it with the same rigor as SLOs.

```python
from dataclasses import dataclass

@dataclass
class ToilItem:
    name: str
    occurrences_per_month: float
    hands_on_minutes: float          # human time only, not wall-clock
    context_switch_minutes: float    # setup and recovery cost per interruption
    grows_with_service: bool         # O(n)?

    @property
    def hours_per_month(self) -> float:
        return self.occurrences_per_month * (self.hands_on_minutes + self.context_switch_minutes) / 60

def payback_months(item: ToilItem, engineering_hours_to_eliminate: float) -> float:
    """Months until the automation project pays for itself (at a flat rate; O(n) toil pays back faster)."""
    return engineering_hours_to_eliminate / item.hours_per_month if item.hours_per_month else float("inf")

def team_ops_fraction(ops_hours: float, total_hours: float) -> float:
    """Measure over a quarter or more. Target ≤ 0.5 (Book ch.5)."""
    return ops_hours / total_hours
```

Rank candidates by hours saved over the next year, **giving more weight to toil that grows with the service**, and against the risk and cost of building the automation.

## How to reduce toil (Workbook ch.6)

Work through these in order:
1. **Reject the toil.** Don't do it. Use SLOs to justify skipping work that doesn't consume error budget.
2. **Engineer it out at the source.** Change the system so the task isn't needed (self-healing, better defaults, removing the knob).
3. **Self-service.** Turn tickets into a tool or API. Aim to move **80–90% of requests** to self-service, with tickets as the fallback.
4. **Automate**, starting with partial automation (human-in-the-loop) and then closing the loop.
5. **Standardize and homogenize** platforms and config, since diversity multiplies toil.
6. **Get management support**: dedicated time, and celebrate toil reduction like a launch.

## Automation hierarchy (Book ch.7)

1. No automation: a human does everything
2. Externally maintained, system-specific automation (a script on someone's laptop)
3. Externally maintained, generic automation
4. Internally maintained, system-specific automation, shipped with the service
5. **Systems that don't need automation**: the service handles it itself, autonomously

The goal is level 5. For example, a database that fails over by itself through consensus is better than a better failover script.

**The value of automation**: consistency (the main value for well-scoped procedures), a platform others can build on, faster repairs (lower MTTR), faster action, and time saved. Judge automation by **competence** (accuracy), **latency** and **relevance** (how much of the real-world process it covers).

**Ads MySQL example** (Book ch.7): manual failover took 30–90 minutes and capped availability at 99%. Automated failover ("Decider") met the under-30-second target 95% of the time, cut operational work by 95%, and freed about 60% of hardware.

## Automation safety rules

Automation operates at machine speed, so its failures do too. (Book ch.7, ch.13; Workbook ch.6)

- **Validate inputs and sanity-check targets.** "Sometimes zero does mean all": Diskerase treated an empty machine list as "every machine". Refuse empty or oversized target sets.
- **Limit blast radius.** Cap how many machines, links or tasks one run may touch, and how many runs may happen per time window.
- **Be idempotent.** Re-running must be safe. Google's Prodtest fixes ran every 15 minutes.
- **Never rely on implicit "safety" signals**, such as assuming something is idle because it is quiet. Check explicitly.
- **Dry-run by default.** Log what would happen, and apply only with an explicit flag.
- **Escalate to a human** when preconditions fail or the automation keeps acting. Don't loop.
- **Give anti-toil automation its own error budget**, and alert when the automation itself fails.
- **The team that runs the service owns its automation.** Code-review it, test it and canary it like production code.

## Example: guarded remediation (replaces "restart when error rate > 5%")

```python
#!/usr/bin/env python3
"""Drain a single unhealthy zone when its error ratio is far above its peers.

Guard rails, following SRE Book ch.7 and ch.13 and Workbook ch.6:
  - dry-run by default
  - acts on at most one zone per run, and never leaves fewer than MIN_HEALTHY_ZONES
  - rate-limited: at most one drain per COOLDOWN, tracked in a state file
  - idempotent: draining an already-drained zone is a no-op
  - escalates (pages) instead of acting when preconditions fail
"""
import argparse, json, sys, time
from pathlib import Path

MIN_HEALTHY_ZONES = 2        # keep N+1 or better after the drain
RATIO_VS_PEERS = 10.0        # zone error ratio must be ≥ 10× the median of the other zones
MIN_REQUESTS = 1_000         # don't act on low-traffic noise
COOLDOWN_S = 3600
STATE = Path("/var/lib/zone-drainer/state.json")

def load_state() -> dict:
    try:
        return json.loads(STATE.read_text())
    except FileNotFoundError:
        return {"last_action": 0, "drained": []}

def decide(zones: dict[str, dict], state: dict) -> tuple[str | None, str]:
    """zones: {name: {"errors": int, "requests": int, "drained": bool}}"""
    serving = {z: m for z, m in zones.items() if not m["drained"]}
    if not serving:
        return None, "ESCALATE: empty target set; refusing to act"          # "zero does mean all"
    if len(serving) <= MIN_HEALTHY_ZONES:
        return None, "ESCALATE: draining another zone would drop below the minimum healthy zones"
    if time.time() - state["last_action"] < COOLDOWN_S:
        return None, "ESCALATE: cooldown active; a human must decide on a second drain"
    ratios = {z: m["errors"] / m["requests"] for z, m in serving.items() if m["requests"] >= MIN_REQUESTS}
    if len(ratios) < 3:
        return None, "NOOP: not enough traffic to compare zones"
    worst = max(ratios, key=ratios.get)
    peers = sorted(v for z, v in ratios.items() if z != worst)
    median_peer = peers[len(peers) // 2]
    if median_peer > 0 and ratios[worst] / median_peer < RATIO_VS_PEERS:
        return None, "NOOP: no single zone stands out; a global problem means rollback, not drain"
    return worst, f"DRAIN {worst}: error ratio {ratios[worst]:.3%} vs peer median {median_peer:.3%}"

def main() -> int:
    p = argparse.ArgumentParser()
    p.add_argument("--metrics", required=True, help="JSON file with per-zone counts (from your TSDB)")
    p.add_argument("--apply", action="store_true", help="actually drain (default is dry-run)")
    args = p.parse_args()
    zones = json.loads(Path(args.metrics).read_text())
    state = load_state()
    zone, reason = decide(zones, state)
    print(reason)
    if zone is None:
        return 2 if reason.startswith("ESCALATE") else 0   # exit 2 → your wrapper pages a human
    if not args.apply:
        print("dry-run: no change made")
        return 0
    # drain(zone)  ← call your load balancer or traffic API here; it must be idempotent
    state["last_action"] = time.time()
    state["drained"] = sorted(set(state["drained"]) | {zone})
    STATE.parent.mkdir(parents=True, exist_ok=True)
    STATE.write_text(json.dumps(state))
    print(f"drained {zone}; open a ticket to root-cause it and undrain it")
    return 0

if __name__ == "__main__":
    sys.exit(main())
```

Why this one and not a restart loop: draining one outlier zone is a generic, reversible mitigation (Workbook ch.9). A global error spike points to a bad change, so it gets a rollback. A blind restart can amplify a cold-cache cascade (Book ch.22).

## Leader-critical state: don't script failover

Scripts like "set primary read-only, `sleep 10`, promote replica, repoint the service" are the ad hoc failover Google calls conceptually unsound. They split-brain when the old primary is merely slow or partitioned. Use a consensus-based system (Raft/Paxos: etcd, ZooKeeper, Patroni with a DCS, a managed database with automatic failover), with fencing and leases. (Book ch.23; see `data-state.md`)

## Anti-patterns

- Counting overhead (meetings, reviews) as toil, or counting toil weekly instead of over quarters
- Ticket-driven processes that could be self-service
- Automation with no dry-run, no limits, no idempotence and no escalation path
- Automation owned by a different team from the one that runs the service
- Remediation loops that restart things on symptoms (restart on 5xx) instead of fixing or rolling back
- Script-based failover of stateful leaders
- "We'll automate it later" with no tracking of toil hours
