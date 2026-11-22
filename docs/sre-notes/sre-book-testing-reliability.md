# SRE Book, Chapter 17: Testing for Reliability (Alex Perry, Max Luebbe)

Source: https://sre.google/sre-book/testing-reliability/. Two equations on the page are images with no alt text. Where this digest gives them as math, they are reconstructed and marked [reconstructed].

## Core principles
- **Testing quantifies confidence.** Past reliability comes from monitoring data. Future reliability is a prediction, and it only holds if the site doesn't change or every change can be described. Testing "demonstrate[s] specific areas of equivalence when changes occur." Each test that passes both before and after a change reduces uncertainty.
- **Coverage buys change budget.** More coverage means more changes before predicted reliability reaches the acceptability limit. If you hit the limit, stop changing and let monitoring data accumulate.
- **Testing and MTTR** (section "Relationships Between Testing and Mean Time to Repair"): a bug that a test catches before it ships, by blocking the push, has *zero MTTR*. The more zero-MTTR bugs you find, the higher the MTBF users see. A higher MTBF lets teams release faster, and new bugs then push velocity back down.
- **Passing proves little, failing proves a lot.** Passing tests don't prove reliability. Failing tests generally prove it's missing.
- **Test cost matters.** Unit tests take milliseconds on a laptop. Full-server tests take "from several minutes to multiple hours" and may need dedicated resources.
- **Not all software is equal.** Life-critical and revenue-critical systems need much more test quality and coverage than short-lived scripts.

## Taxonomy (section "Types of Software Testing")
- **Traditional** tests run offline. They form a hierarchy (Fig. 17-1):
  - **Unit:** a class or function in isolation. Doubles as a specification.
  - **Integration:** assembled components, using dependency injection (Dagger) to mock things like a stateful DB.
  - **System:** the largest test of an *undeployed* system.
    - **Smoke** (aka sanity): short-circuits more expensive testing.
    - **Performance:** catches gradual slowdown, e.g. 8 GB growing to 32 GB, or 10 ms growing to 50 ms, then 100 ms.
    - **Regression:** a "gallery of rogue bugs."
- **Production** tests run against the live system. They are similar to black-box monitoring and are also called "black-box testing."
  - **Configuration test:** compares what production actually runs against the checked-in config file. It is non-hermetic by design. It gets harder when the config relies on defaults built into the binary, goes through a preprocessor such as bash into flags, or depends on a shared runtime's release schedule.
  - **Stress test:** finds limits, e.g. how full a DB can get before writes fail, or how much QPS a server takes before it overloads. Components often fail catastrophically rather than degrade gracefully.
  - **Canary:** "conspicuously absent" from the list of tests. It is "structured user acceptance," not a real test.

## Concrete numbers, thresholds, formulas
- **Canary rollout rule of thumb (fn 89):** start at **0.1%** of user traffic, then grow by an order of magnitude every **24 hours** while varying geography: day 2 **1%**, day 3 **10%**, day 4 **100%**.
- **Canary variance formula:** **C = R·K^U**. The page's plain-text rendering shows "CU = RK", which is a superscript lost in extraction.
  - C = cumulative number of reported variances; R = rate of reports; U = order of the fault; K = the period over which traffic grows by a factor of *e*, "or 172%".
- **Fn 90:** with 24 h of exponential growth from 1% to 10%, K = 86400 s / ln(10) ≈ 37,523 s [reconstructed], or "about 10 hours and 25 minutes."
- **Fault order U** (in the big-O sense):
  - U=1: the request hit simply broken code.
  - U=2: the request randomly damages data that a later request may see.
  - U=3: the damaged data is also a valid identifier to a previous request.
  - "Most bugs are of order one."
- **Flakiness math (section "Testing Deadlines"):** take a service with **21,000** tests, run before and after a patch, so 42,000 results. A **1 in 100** false-reject rate is tolerable; **1 in 10** draws complaints. The 42,000th root of 0.99 ≈ 0.99999976 [reconstructed], so each test "must run correctly over **99.9999%** of the time."
- In the migration discussion, engineers "would likely accept the test system giving the wrong answer 1 time in 10 or so."
- **Interactive tests** run in a small container for a few seconds. Multi-binary orchestration tests are *batch* tests. Informal deadline: return results before the engineer's next context switch.

## Recommended practices & procedures
- **Starting from low coverage** (section "Creating a Test and Build Environment"):
  - Stack-rank components by importance.
  - Find the mission- or business-critical code first. Billing is the example: it's critical and often cleanly separable.
  - Protect the APIs other teams integrate against.
  - Smoke-test every release.
  - Convert every reported bug into a test that fails at first.
- **Infrastructure:**
  - Versioned source control comes first, then a continuous build that runs tests on every submit and notifies engineers the moment a change breaks the build.
  - A broken build means dropping everything else, for four reasons: later changes make the fix harder, the team slows down, nightly and weekly cadences lose value, and emergency releases (e.g. a security fix) get harder.
  - "Stability actually drives agility."
  - Use Bazel-style dependency graphs to run tests only for changed code.
  - Set explicit coverage goals and deadlines rather than saying "We need more tests."
- **Barrier defenses for risky tools:**
  - A separate tool puts a barrier in the replication config so the replica fails its health check.
  - The risky software checks for the barrier at startup and only touches unhealthy replicas.
  - The black-box health validator removes the barrier.
- **Disaster tools (section "Testing Disaster"):** design them offline with five properties: **offline, checkpoint, loadable, barrier, clean start.** Breaking any of them makes confidence much harder.
- **Statistical tests:**
  - Lemon (fuzzing), Chaos Monkey and Jepsen aren't repeatable, so log the RNG seed or the actions.
  - Refactor the log into a release test and replay it a few times. The rate of non-failure on replay tells you how hard it will be to show the fix later.
  - Escalate severity if replays show worse failures.
- **Config files (section "Expect Testing Fail"):**
  - Put each config file in exactly one of two classes: (a) the MTTR-lowering file, edited only on failure, or (b) the frequently changed file, which must be treated like an application release.
  - Give each file enough test coverage for routine edits.
  - Delay edits slightly before releases while release testing runs.
  - Provide a *noisy* break-glass that files a bug.
- **Break-glass:** don't disable tests. Keep them running, link the early push to the pending tests, back-annotate the push with any failures, and boost those tests' priority.
- **Config syntax** (in increasing order of preference):
  - Interpreted language (e.g. Python): no upper bound on load time, so add deadline checks.
  - Custom syntax: needs coverage from scratch.
  - YAML with `safe_load`: bounded load time, but schema checks are unbounded.
  - Protocol buffers: the schema is checked at load time and runtime is bounded.
- **Production probes:**
  - Use three request sets: known bad, known good that can be replayed against prod, and known good that can't. Use them as integration tests, release tests and monitoring probes.
  - The updater keeps all four old/new probe × old/new app combinations running and rolls back on errors.
  - Make probes part of the readiness check.
- **Fake backends:** the peer team cuts a fake-backend release alongside its real backend. Monitoring tracks every version pair across an interface. Rollout automation blocks the production rollout until problematic combinations can't occur.
- **Defense in depth:** a tool that detects misbehavior should *report* it, not auto-remediate. Example: a home-directory tool notices a half-parsed /etc/passwd and must not delete user data.
- **Reliability budget:** segment it by functionality, or more conveniently by team. SRE gets its own budget and release-rate ceiling.

## Anti-patterns
- Shipping obviously broken software ("among the most cardinal sins").
- Trying to write unit tests for every function when coverage is near zero.
- Keeping test infrastructure segregated from production config. In the SRE model this prevents architecture change "because there is no way to eliminate migration risk."
- A break-glass that silently disables release testing.
- Frequently changed config files with weaker coverage than the application. Such a file "will dominate site reliability in a negative way."
- Assuming a rerun of a randomized test proves a fix (fn 98: kills and traffic aren't serialized).
- Assuming hermetic test passes mean operational correctness when the config test fails.

## Checklists
- **Where to start testing:** priority ranking, critical functions, APIs other teams integrate against.
- **Disaster tool properties:** offline, checkpoint, loadable, barrier, clean start.
- **Probe sets:** known bad, known good that can be replayed, known good that can't.

## Verbatim quotes
- "If you haven't tried it, assume it's broken." (epigraph)
- "Passing a test or a series of tests doesn't necessarily prove reliability. However, tests that are failing generally prove the absence of reliability."
- "A canary test isn't really a test; rather, it's structured user acceptance."
- "Instead of repeating the ambiguous refrain 'We need more tests,' set explicit goals and deadlines."
- "Therefore, the monitoring probe running in production is a configuration that wasn't previously tested."
