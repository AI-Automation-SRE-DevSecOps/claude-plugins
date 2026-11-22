# Release Engineering, Canarying and Configuration

Sources: SRE Book ch.8 "Release Engineering", ch.9 "Simplicity", ch.17 "Testing for Reliability", ch.27 "Reliable Product Launches at Scale", App. B. SRE Workbook ch.14 "Configuration Design and Best Practices", ch.15 "Configuration Specifics", ch.16 "Canarying Releases", App. C.

Change is the main risk. About **70% of outages are caused by changes** (Book ch.1, Workbook App. B). Binary pushes trigger **37%** of outages and configuration pushes **31%** (Workbook App. C). Make every change **gradual, observable and reversible**.

## Release engineering principles (Book ch.8)

1. **Self-service**: teams release on their own using common tooling.
2. **High velocity**: frequent, small releases, so each one carries fewer changes and is easier to debug. Some teams use "push on green": deploy every build that passes all tests.
3. **Hermetic builds**: the same revision produces the same output on any machine. Build tools are versioned with the code, and nothing is fetched from outside.
4. **Enforcement of policies and procedures**: code review, access control on who can release, and an audit trail.

- **Branching**: cut the release branch from a revision on mainline and never merge it back. Fixes land on mainline first, then get cherry-picked into the release.
- **Testing**: cut from the last fully passing continuous build. Re-run tests on the release branch, because cherry-picks create code that never existed on mainline.
- **Rollout speed depends on risk**: anywhere from hourly automatic pushes to multi-day, region-by-region rollouts.
- **Keep releases small**, like gradient descent: small steps make it easier to see which change caused what. (Book ch.9)

## Canarying (Workbook ch.16)

- **Definition**: "a partial and time-limited deployment of a change in a service and its evaluation." Compare a canary population against a **control** population running the old version, at the same time.
- **Canary one change at a time.** Google "strongly advise[s]" against overlapping canaries.
- **Error-budget cost scales with exposure**: a defect hitting 20% of requests on a 5% canary appears as about a 1% overall error rate. Keep canaries small enough that a bad one costs an acceptable share of the budget.
- **Metrics**: start from SLIs, using "perhaps no more than a dozen", such as HTTP status ratios and latency. Prefer user-facing signals over CPU or memory. **Leave out 4xx**, since clients cause them.
- **Break every metric down by canary vs control.** Use evaluation windows no longer than the canary itself. An errors-per-hour metric is useless for a 30-minute canary.
- **Before/after comparisons are risky** because time is a big source of noise. **Blue/green** deployment is effectively before/after unless traffic shifts gradually.
- **Also check absolute SLOs.** Canary and control aren't perfectly isolated, so a canary can harm the control group too.
- **Pipelines and batch jobs**: the canary must last at least one full work unit.
- **No fixed numbers.** The books give no universal canary size or duration. Choose them from traffic volume, so there are enough requests for statistical significance, and from how long the change takes to show problems.

### Staged rollout

- Book ch.17 fn.89 gives one example: start at **0.1%** of traffic, then grow tenfold every 24 hours to 1%, 10%, 100%. Workbook ch.13 uses a **1% dry run, then about 1%, 10%, 50%, 100%** for pipelines. Treat both as examples.
- Rolling out to 1% of users first turns a 26-second budget exhaustion (99.999%) into 43 minutes. (Workbook ch.5)
- Stage by **failure domain** (one cell or zone, then one region, then the rest), and wait long enough at each stage to see effects.
- **Avoid Mondays and Fridays** (launch checklist, Book App. E), and avoid rollouts at traffic peak.
- Every stage needs an **automatic stop or rollback** if canary metrics degrade against control.
- **Roll back first, debug afterwards.** (Book App. B)

## Feature flags and launches (Book ch.27)

- Ship **dark** behind a flag, then ramp to "usually between 1 and 10 percent" of users, then expand.
- Each change must be revertible on its own. Avoid lockstep releases and API-incompatible changes.
- **Dark launch**: send a copy of real traffic to the new path and discard its responses, to test load and correctness. (Book ch.32)

## Configuration design (Workbook ch.14, ch.15)

- Configuration is "a human-computer interface for modifying system behavior". The ideal amount of configuration is **none**: let the system work out what it can.
- **Design it around users**: fewer knobs, expressed as goals ("latency-sensitive") rather than implementation details ("thread pool = 64").
- **Use safe defaults** (static or dynamic, such as threads = cores, dry-run on). Turn mandatory questions into optional ones, and give power users overrides.
- **Separate code from data.** Infrastructure should consume static data (protobuf, JSON, YAML) generated from a higher-level config language. Use an existing language (Jsonnet, CUE, Dhall, HOCON) rather than general-purpose Python, Ruby or Lua.
- **Evaluation must be hermetic and side-effect free**: evaluate, let people inspect the result, then apply. Google commonly evaluates at **build time**.
- **Validate semantics, not just syntax**: check the schema and domain rules, **never ignore unknown fields**, and run the same checks before commit.
- **Shape**: key objects by name rather than putting them in arrays, and group them by function.
- **Every config needs an owner and a version**, even when it's set through a UI or API. Log both when it changed and when it was applied.
- Untrusted config should run in a separate process with resource limits. A non-Turing-complete language doesn't prevent resource exhaustion.

### Safe config change (Workbook ch.14)

Config changes need the same three properties as binary changes:
1. **Gradual rollout**
2. **Rollback**
3. **Automatic rollback, or at least a stop**, if operators lose control

Book App. B adds: **on a bad config, keep serving with the previous config and alert**. Validate syntax and semantics, and alert on a sudden drop in config size.

### Config testing (Book ch.17)

Each config file should be treated one of two ways. Either it is a lever that changes only MTTR and can be pushed fast, with tests, or it is released through the same canary pipeline as a binary. Prefer schema-checked formats (protobuf) that load in bounded time.

## Checklist

- [ ] Hermetic, reproducible builds; release branches cut from green mainline; cherry-picks re-tested
- [ ] One change per canary; canary and control running at the same time with metrics split by population
- [ ] Canary metrics are SLIs (5xx ratio, latency thresholds), at most about 12, no 4xx
- [ ] Canary cost within the error budget (exposure × expected defect rate)
- [ ] Staged rollout by failure domain, with automatic halt or rollback
- [ ] A rollback for every change, tested before risky rollouts
- [ ] Config: versioned, owned, schema-validated before commit, rolled out gradually, last-known-good kept on failure
- [ ] Feature flags for user-visible changes; ramp from 1–10%
- [ ] No pushes at peak or on Fridays without explicit risk acceptance
