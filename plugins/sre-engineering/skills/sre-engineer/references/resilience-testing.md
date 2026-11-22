# Testing for Reliability, Disaster Testing and Chaos

Sources: SRE Book ch.13 "Emergency Response", ch.17 "Testing for Reliability", ch.22 "Addressing Cascading Failures", ch.26 "Data Integrity", ch.28 (Wheel of Misfortune), ch.33 "Lessons Learned from Other Industries". SRE Workbook ch.8, ch.9, ch.10.

## Principles (Book ch.17)

- Passing tests don't prove a system is reliable, but failing tests usually prove it isn't.
- **A test that blocks a bad push has an MTTR of zero.** The bug never reaches users. More such tests raise the MTBF that users see.
- Test at several levels: unit, integration and system. System tests include smoke, performance and regression tests. Production tests include canaries, config tests, stress and load tests, and disaster tests.
- **Canaries are "structured user acceptance"**, not a substitute for tests (see `release-config.md`).
- **Flaky tests**: with 21,000 tests run before and after a change, keeping false rejections to 1 in 100 requires each test to pass more than 99.9999% of the time. Flakiness hides real failures.
- **Break-glass**: when you must push past failing tests, keep the tests running, attach the failures to the push, and file a bug. Never disable testing quietly.
- Disaster tools should work **offline**, support **checkpoints**, be **loadable**, use **barriers**, and allow a **clean start**.

## Disaster and Recovery Testing (DiRT) (Book ch.33)

- Google runs DiRT **every year**. It deliberately causes real outages in production systems and processes.
- Goals: confirm systems and people react as expected, find unknown weaknesses, and harden against uncontrolled outages.
- Test the technical systems (failover, restores, capacity loss) and the human systems (escalation, communication, access, documentation).
- Ask "what if…?" questions ahead of time, and **test while your best people are present**. (Book ch.13)
- **Require a tested rollback before any large-scale test.** (Book ch.13)
- Keep **out-of-band communication and access** working and practiced.

## Chaos and fault-injection experiments

Google's books describe DiRT and production fault testing rather than "chaos engineering" tooling, but the same rules apply:

1. **Hypothesis**: state the expected steady state in SLI terms, for example "availability SLI stays ≥ 99.9% while one zone is down".
2. **Blast radius**: start with the smallest scope (one task, one canary cell, a staging or low-criticality slice) and grow only after passing. Never start at "50% of production pods".
3. **Error-budget gate**: run only when enough budget remains to absorb a failure, and estimate the budget the test could spend.
4. **Abort criteria**: SLI-based, automatic, and checked often. Abort on SLO burn rate, not on an arbitrary error-rate number.
5. **Tested rollback**: rehearse the stop and undo action before injecting.
6. **Timing**: during **staffed working hours**, with owners and on-call aware. Avoid peak traffic and change freezes.
7. **Observe and write up**: record timeline, observations and action items in the postmortem format, including when the test "passed".

```python
import time
from dataclasses import dataclass, field
from datetime import datetime, timezone
from typing import Callable

@dataclass
class Experiment:
    name: str
    hypothesis: str                      # in SLI terms
    blast_radius: str                    # e.g. "1 of 12 tasks in cell us-east1-b"
    max_budget_spend: float              # fraction of the 4-week budget the test may use, e.g. 0.02
    abort_burn_rate: float = 14.4        # abort if the short-window burn rate exceeds this
    staffed_hours: tuple[int, int] = (10, 16)   # local hours when owners are present
    observations: list[str] = field(default_factory=list)

def preflight(exp: Experiment, budget_remaining: float, now: datetime,
              rollback_tested: bool, change_freeze: bool) -> list[str]:
    problems = []
    if not rollback_tested:
        problems.append("rollback not rehearsed")                      # Book ch.13
    if change_freeze:
        problems.append("change freeze / error budget policy in effect")
    if budget_remaining < 2 * exp.max_budget_spend:
        problems.append("not enough error budget to absorb a failed test")
    start, end = exp.staffed_hours
    if not (start <= now.hour < end) or now.weekday() >= 5:
        problems.append("outside staffed working hours")              # run while people are present
    return problems

def run(exp: Experiment, inject: Callable[[], None], undo: Callable[[], None],
        burn_rate_5m: Callable[[], float], duration_s: int, poll_s: int = 10,
        sleep: Callable[[float], None] = time.sleep) -> str:
    inject()
    try:
        for _ in range(duration_s // poll_s):
            br = burn_rate_5m()
            exp.observations.append(f"{datetime.now(timezone.utc).isoformat()} burn={br:.1f}")
            if br > exp.abort_burn_rate:
                return "ABORTED: hypothesis falsified (burn rate exceeded); write it up"
            sleep(poll_s)
        return "PASSED: hypothesis held"
    finally:
        undo()      # always undo, then verify the SLIs recover before closing out
```

## Wheel of Misfortune (Book ch.28, Workbook ch.8)

- A role-play of a past incident: a **game master** runs the scenario and **two players** act as primary and secondary on-call.
- Run it **weekly for 30–60 minutes**, build scenarios from real postmortems, and record the sessions.
- It exercises diagnosis, incident roles, communication and playbooks without touching production.

## Recovery and data restore testing (Book ch.26)

- "You only know that you can recover your recent state if you actually do so." **Automate end-to-end restore tests** and alert when they fail.
- Measure the time to restore against a data-recovery SLO.
- Test the human procedure as well as the tooling.

## Load and overload tests (Book ch.22)

- Test **past the breaking point** to see the failure mode. Check that the system degrades gracefully, sheds load, recovers without help, and how far load must drop before it does.
- Test gradual increases and sudden spikes.
- In production: kill tasks, lose a cluster, blackhole backends. Confirm that clients back off and that broken non-critical backends don't take the service down.

## Anti-patterns

- Chaos experiments with large initial blast radius, no tested rollback, or abort thresholds unrelated to the SLO
- Running fault injection outside staffed hours to "avoid impact". This removes the people who need to learn and respond.
- Treating a canary as the only test
- Disabling flaky tests instead of fixing them
- Backups that are never restored
- Drills without a written outcome and action items
