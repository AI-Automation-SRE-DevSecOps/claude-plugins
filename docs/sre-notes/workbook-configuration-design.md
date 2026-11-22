# SRE Workbook Ch. 14: Configuration Design and Best Practices

Source: https://sre.google/workbook/configuration-design/ (Štěpán Davidovič with Murphy, Kalt, Beyer). Ch. 15 "Configuration Specifics" covers Jsonnet and mechanics in more depth.

## Core principles

- **Definition** ("What Is Configuration?"): "a human-computer interface for modifying system behavior". It gives a low-overhead way to change behavior without a rebuild and redeploy. A system has three components: the software, the data set, and the configuration. The boundaries between them blur (mod_lua, XMonad's Haskell config, stored SQL procedures).
- **Config quality is reliability** ("Configuration and Reliability"): one option can change behavior drastically (a bad firewall rule locks you out). Config is often untested or untestable, and it gets changed under incident pressure. The analogy is early aircraft controls, where operator failures were frequent "regardless of pilot skill or experience."
- **Separate philosophy from mechanics**: philosophy covers structure, the level of abstraction, and diverging use cases, independent of language. Mechanics covers language, deployment, and how config interacts with other systems.
- **Ideal is no configuration** ("Configuration Philosophy"): the system infers settings from deployment, workload, or existing config. The ideal is rarely reachable but sets the direction: "away from a large number of tunables and toward simplicity."
- **Configuration asks users questions**: Google favors the **user-centric view** (fewer knobs, because answering them is a chore) over the **infrastructure-centric view** (more knobs, so you can tune "to perfection"). This requires user research. Limited options can drive *better* adoption because the software works "out of the box."
- **Questions should be close to user goals**: let users state goals ("hot green tea"), not implementation steps (water volume, temperature, steep time). The system can then improve how it meets those goals over time. Example: Kubernetes and Mesos run your job without asking which machine.

## Concrete numbers / thresholds / tables

The chapter has **no numeric thresholds, formulas, or tables**. Its only figures:
- Canary Analysis Service: "about a month" spent reducing mandatory questions and finding good optional defaults (Conclusion).
- Semantic validation should catch a config needing "a thousand times more RAM" because of a units mistake.
- Gradual rollout lets you abort "before causing a 100% outage."
- Fig. 14-2: a config interface (language or web UI) compiles to a static config-data layer. The web UI also displays the current config, so the flow is bidirectional.

## Recommended practices

**Mandatory vs. optional questions**
- *Mandatory* questions are needed for any functionality (e.g., who to charge). *Optional* questions improve quality (e.g., worker-process count). Minimize mandatory questions. The easiest way is to convert them to optional ones with safe defaults (e.g., dry-run by default).
- **Dynamic defaults**: threads = cores in the machine or container, and Java heap = container memory. Keep them overridable.
- If a *significant portion* of users have trouble with a dynamic default, improve the logic broadly *without adding knobs*. If only a *small fraction* do, they should set the option manually.
- Most users keep defaults, so a wrong default does "a lot of harm" (organ-donor default study cited).
- Add knobs only for a real need. Remove optional questions with no clear use case. With inheritance, leaf configs should be able to revert to the default.

**Escaping simplicity** ("Escaping Simplicity")
- Don't force everyone to the power user's lowest common denominator. Use high-level defaults plus **optional overrides** ("green tea" + "steep for five minutes").
- Optimize **the sum of hours spent configuring across the organization**. That includes decision paralysis, fixing wrong turns, and slower change from low confidence. Fewer but harder steps for complex cases is fine if the common case gets much easier.
- If more than a small subset of users need complex config, you probably misidentified the common use cases. Do more user research.

**Separate configuration and resulting data** ("Mechanics of Configuration")
- Config is code *and* data, kept separate. Infrastructure consumes **plain static data** (Protocol Buffers, YAML, JSON). Users write a higher-level interface (Python DSL, Lua, Jsonnet, or a web UI) that *compiles* to that data, for example through a post-commit hook or a periodic process (which adds delay).
- Analyze the static data, e.g., load JSON into PostgreSQL to see which parameters are used and by whom.
- Store ingestion metadata, such as the source file path and language, so you can trace authors.
- Plain JSON as the interface is acceptable. Avoid tight coupling between the interface format and the internal format.

**Tooling** ("Importance of Tooling")
- **Semantic validation**, not just syntax (catch typo'd paths and unit errors). Ask whether each misconfiguration could be caught "at the moment the user commits."
- Editor syntax highlighting, a **linter** (Pylint), and an **autoformatter** (clang-format, autopep8). In large orgs, add deprecation annotations and automatic rewriting tools.

**Ownership and change tracking**
- Every config snippet has a **clear owner**, ideally one owner per file or directory.
- **Version all config**, including config entered through a web UI or API. Consider coupling config to software versions.
- Log both config **changes and their application**, because a commit does not always mean the config was applied. This enables confident rollback and lets you notify affected parties. It complements blameless postmortems.

## Checklist: safe configuration change application

A safe change has **three properties**:
1. Can be **deployed gradually**, not all-or-nothing. No global all-at-once push. Kubernetes rolling updates are the cited example (see Ch. 16 Canarying).
2. Can be **rolled back**. This is faster and more trustworthy than a temporary patch.
3. **Automatic rollback (or at minimum, the ability to stop progress) if the change leads to loss of operator control.** Example: a screen-resolution confirm countdown, versus an admin firewalling themselves out.

Note: config must be **hermetic** to roll forward and back. Config in version control that references data on a network filesystem is not hermetic. The same principles apply to binary and data-set pushes.

## Anti-patterns

- Infrastructure-centric "more knobs is better." Many mandatory questions. Treating small steps as free.
- Knobs without a real use case. Wrong defaults.
- Making simple use cases work at the power users' level of complexity.
- Infrastructure consuming the high-level language. Coupling the interface format to the internal format.
- Syntax-only validation. Catching errors after submission.
- Unversioned UI/API config. Unclear ownership.
- Global pushes. Patching forward instead of rolling back. Non-hermetic config.
- Config designed as "a side effect of system implementation."

## Verbatim quotes

- "our ideal configuration is no configuration at all."
- "Experience shows that most users will use the default, so this is both a chance and a responsibility."
- "having *both* code and data, but separating the two, is optimal."
- "In order to be able to roll forward and roll back configuration, it must be hermetic."
- "Configuration design carries aspects of both API and UI design and should be purposeful"
