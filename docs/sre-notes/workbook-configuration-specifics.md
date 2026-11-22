# SRE Workbook, Chapter 15: Configuration Specifics

Source: https://sre.google/workbook/configuration-specifics/. Authors: Dave Cunningham and Misha Brukman, with Christophe Kalt and Betsy Beyer. This is the hands-on companion to Ch. 14, "Configuration Design and Best Practices". Jsonnet is used for illustration only. The chapter says "There is no dominant configuration language at Google."

## Core principles

- **Two kinds of config toil** (section "Configuration-Induced Toil"):
  - *Replication toil*: "the mundane task of managing configuration replicated across a system." It is common in microservice architectures, not only at large organizations.
  - *Complexity toil*: dealing with "the emergent and sometimes undesirable behaviors of complex automation." It "typically materializes in larger organizations (10+ engineers) and compounds with growth." Tackle it early.
- **Strategy order** ("Reducing Configuration-Induced Toil"): (1) remove configuration altogether where the application can decide for itself, for example defaults taken from machine info or values that scale with load. This is "always your best option" when possible. (2) If replication toil is the problem, automate or adopt a config language. (3) Use operating best practices to keep complexity toil down.
- **Three critical properties** ("Critical Properties and Pitfalls"; repeated in the Conclusion as the "minimum"):
  1. **Tooling**: linters, debuggers, formatters, IDE integration.
  2. **Hermetic evaluation**, for rollbacks and replayability. Config must "generate the same config data regardless of where or when they execute."
  3. **Separation of config and data**.
- Treat configuration as a programming-language problem and design the language on purpose.

## Concrete numbers, thresholds and facts (as written)

- Complexity toil appears in "larger organizations (10+ engineers)".
- Resource-exhaustion examples ("Guarding Against Abusive Configuration"):
  - Nonterminating: `local f(x) = f(x + 1); f(0)`
  - Unbounded memory: `local f(x) = f(x + [1]); f([])`
  - Terminating but practically infinite (2^100 work): `local f(x) = if x == 0 then [] else [f(x - 1), f(x - 1)]; f(100)`
- Configuration changes "tend to dominate outage root causes over time" (cross-reference to Appendix C, "Results of Postmortem Analysis").
- A YAML stream is a set of documents separated by `"---"`. YAML anchors (Spec §6.9.2) are "rarely useful in practice and are not supported by Kubernetes."
- Versioning by directory path, e.g. `import 'ksonnet.beta.2/k.libsonnet'`.

### Table: when to evaluate configuration ("When to Evaluate Configuration")

| Stage | Pros | Cons |
|---|---|---|
| **Very early**: check in generated JSON | Reviewer sees concrete diff (a refactor should not change the JSON); line annotations at both levels for auditing; no Jsonnet at runtime | JSON may be unreadable; may be too large or contain secrets; merge conflicts when many Jsonnet files converge on one JSON file |
| **Middle**: evaluate at build time, embed in the release artifact. **Google commonly favors this.** | Controls runtime complexity, binary size and risk without regenerating JSON per PR; no desync between Jsonnet and JSON | More complex build; harder to see the concrete change in review |
| **Late**: evaluate at runtime (link library) | Simpler; can evaluate user-supplied code | Bigger footprint and risk exposure; bugs found at runtime ("too late"); untrusted code needs special care |

Examples: ksonnet (a client CLI) evaluates in-process, which is safe because it runs on the author's machine. Box.com's Git hooks act on checked-in JSON. Helm and Spinnaker have to evaluate on the server at runtime.

## The five pitfalls (anti-patterns)

1. **Failing to recognize configuration as a programming language problem.** Language features creep into data-only formats. Examples: a `count` attribute on a VM schema, string interpolation rules used in place of real expressions, and YAML + Jinja. Each format is fine for its own purpose; combined, they become hard for humans and tools.
2. **Designing accidental or ad hoc language features.** These end up "more complex and usually have less expressive power than their formally designed equivalents", with gotchas. Plan for programming constructs at the design phase.
3. **Building too much domain-specific optimization.** A small user base means tooling never gets justified and there is no Stack Overflow help.
4. **Interleaving configuration evaluation with side effects.** Examples are changing external systems or reading DNS, VM IDs or the latest build versions. This violates hermeticity. The correct order is to evaluate, then expose the data for analysis, and only then apply side effects.
5. **Using a general-purpose scripting language (Python, Ruby, Lua).** It is heavyweight, needs intrusive sandboxing for hermeticity and security, and maintainers may not know the language.

**Recommendation:** use an existing configuration DSL such as HOCON, Flabbergast, Dhall or Jsonnet. If it is too powerful, restrict it with an in-house style guide.

## Recommended practices

### Integrating a configuration language
- For non-native formats, represent the data in the language and write or reuse a serializer (the Jsonnet stdlib covers INI and XML). Use string templating, for example for Bash, only "as a last resort."
- Drive several applications from one evaluation. Example: Nginx and Terraform firewall configs that define the port once.
- Templates versus functions: functions only allow parameterization, while template overrides can change any field. That override is the "escape hatch." Hidden fields use `::`. `error 'Needs tier'` acts as an abstract method.
- **Conversion workflow:** YAML variant → JSON → Jsonnet formatter → add abstraction by hand.

### Designing in-house applications for config ("Integrating Custom Applications")
- Consume **a single pure-data file** and let the language split sources through imports.
- Key named collections **by name** (`animals.cat`), not as arrays of `{ "name": ... }` (brittle `animals[0]`).
- **Don't group entities by type at the top level.** Group by function, for example `pot_assembly1: {pot, lid}`.
- Keep the data representation simple. Don't embed language features in it and don't worry about verbosity. Avoid custom string interpolation unless the action happens after generation (alerts, handlers).
- Fix naming inconsistencies in the model, not in the language layer.

### Validation
- Validate generated data **immediately after evaluation**. Syntactic checks alone "won't find many bugs." After schema validation, check domain properties: required fields are present, referenced files exist, values are in range. JSONschema and protobuf deserialization both work for this.
- **Do not ignore unrecognized field names**, because they may be typos. Run the same validation in a **precommit hook**.

### Operating ("Effectively Operating a Configuration System")
- **Versioning**: for a breaking change, update every client globally or version the library by directory. Consumers left on deprecated versions incur technical debt.
- **Source control** gives history, authorship, rollback and code review. **Tooling**: run formatters and linters through editor and precommit hooks.
- **Testing**: unit-test template libraries with `assert` or `std.assertEqual`, which shows the mismatched values.

### Guarding against abusive configuration
- Making the language non-Turing-complete does **not** stop resource exhaustion. Even XML and YAML have such programs.
- Trusted CLI use carries low risk (Ctrl-C works). A service that evaluates untrusted code in a request handler (Helm, Spinnaker) risks DoS.
- **Mitigation**: run evaluation in a separate process under `ulimit` by forking the CLI rather than linking the library, so it fails safely and tells the user. Use the Go implementation of Jsonnet for extra defense against C++ memory exploits.

## Checklist (derived)

- [ ] Can the application remove or derive the config itself?
- [ ] Is the language an existing DSL with tooling, hermetic, and free of side effects?
- [ ] Is the app input a single pure-data file, with name-keyed objects grouped by function?
- [ ] Is there schema plus domain validation, with unknown fields rejected and checks also run in precommit?
- [ ] Are libraries versioned and unit-tested, and was the evaluation stage chosen deliberately?
- [ ] Is untrusted evaluation sandboxed (separate process, ulimit)?

## Short quotes

- "If you're not intentionally designing a language, then it's highly unlikely the 'language' you'll end up with is a good one."
- "first evaluate the config, then make the resulting data available to the user to analyze, and only then allow for side effects."
- "configuration changes tend to dominate outage root causes over time in a system"
- "do not ignore unrecognized field names, as they may indicate a typo"
