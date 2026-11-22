# SRE Book, Chapter 8: Release Engineering

Source: https://sre.google/sre-book/release-engineering/. Written by Dinah McNutt; edited by Betsy Beyer and Tim Harvey.

The chapter defines release engineering as "building and delivering software". SREs need binaries and configs built "in a reproducible, automated way" so that releases are repeatable and not "unique snowflakes". Changes to the release process "should be intentional, rather than accidental."

## Core principles (section "Philosophy": four principles)

1. **Self-Service Model.** Teams must be self-sufficient to work at scale. Product teams decide how often and when to release. Releases are "truly automatic" and need engineers only "if and when problems arise."
2. **High Velocity.** "Frequent releases result in fewer changes between versions," which makes testing and troubleshooting easier. There are two variants:
   - Hourly builds, with the version to deploy *selected* from that pool based on test results and features.
   - "Push on Green": deploy *every* build that passes all tests [Kle14].
3. **Hermetic Builds.** The same product built at the same revision on different machines should give identical results. Builds are insensitive to whatever is installed on the build machine. They depend on known versions of build tools (compilers) and dependencies (libraries), and must not rely on services outside the build environment. Build tools are versioned with the project's repo revision, so a cherry-pick rebuild of last month's release uses last month's compiler.
4. **Enforcement of Policies and Procedures.** Layered security and access control cover who may do each gated operation (see the checklist below). Almost all changes need code review. The automated release system archives a report of all changes in a release next to the build artifacts, which speeds up troubleshooting.

## Concrete numbers, thresholds, tables

The chapter has almost no hard numbers. Do not attribute specific thresholds to it. What it does say:
- "Some teams perform hourly builds."
- Rapid "can handle thousands of release requests simultaneously" because it runs on production infrastructure (Borg).
- Rollout durations scale with risk: Sisyphus can update all jobs immediately or roll out "over a period of several hours". Sensitive infrastructure may roll out "over several days".
- A large user-facing service starts in one cluster and "expand[s] exponentially until all clusters are updated". No multiplier is given.
- A typical canary is "starting a few jobs in our production environment after the completion of system tests". No count or duration is specified.
- There are no formulas or tables. The only figure is Figure 8-1, a simplified view of the Rapid architecture.

## Recommended practices and procedures

**Building (Blaze, open-sourced as Bazel).** Build targets and their dependencies are declared, and Rapid passes project flags such as a unique build ID. *Every binary supports a flag that shows build date, revision number, and build ID*, so any binary can be traced back to how it was built.

**Branching.** All code goes into mainline. Major projects branch at a specific revision and **never merge the branch back into mainline**. Bug fixes land on mainline *first* and are then cherry-picked into the release branch. Because unrelated mainline changes are never picked up, the exact contents of each release are known.

**Testing.**
- A continuous test system runs unit tests on mainline at every submitted change.
- Continuous-build test targets should be the *same* targets that gate the release.
- Releases should be cut at the revision of the *last continuous test build that passed all tests*.
- Unit tests are re-run on the release branch, with an audit trail, because cherry-picks can produce code "that doesn't exist anywhere on the mainline."
- An independent environment runs system-level tests on packaged artifacts, launched manually or from Rapid.

**Packaging (Midas Package Manager, MPM).** Packages are named (e.g. `search/shakespeare/frontend`), versioned with a unique hash, and signed. Rapid applies a build-ID label so that package name plus label is a unique reference. Lifecycle labels (`dev`, `canary`, `production`) *move* automatically when applied to a new package.

**Rapid (release system).** It is configured with **blueprints**: build/test targets, deployment rules, and owners, protected by role-based ACLs. Workflows run serially or in parallel and can launch other workflows. Typical flow:
1. Create a release branch from the integration revision, which often comes from the continuous test system.
2. Compile with Blaze and run unit tests, often in parallel, in dedicated environments rather than in the Rapid Borg job.
3. Use the artifacts for system tests, then canary deployments.
4. Log each step and generate a report of all changes since the last release.

Each cherry-pick request is approved or rejected individually.

**Deployment.** Rapid handles simple deployments directly by updating Borg jobs. Complex ones go through **Sisyphus**, the SRE-built rollout framework. It provides extensible Python classes and a dashboard, and selects the MPM version by build label. Principle: "fit the deployment process to the risk profile of a given service." Examples:
- Dev/pre-prod: build hourly and auto-push when tests pass.
- Large user-facing: one cluster, then exponential expansion.
- Sensitive infrastructure: several days, interleaved across geographic regions.

## Configuration management (section "Configuration Management")

Config changes are "a potential source of instability." **All** schemes keep config in the primary source repo and require strict code review. There are four models:

| Model | Pro | Con |
|---|---|---|
| Use the mainline for configuration | Decouples binary and config releases; simple | Skew between checked-in and running config, because jobs must be updated to pick up changes |
| Config + binaries in the same MPM package | One package to install | Less flexible; binary and config tightly bound |
| Separate MPM "configuration packages" | Hermetic; reconstruct config by build ID; change config without rebuilding the binary (e.g. cherry-pick a flag change from `first_folio` to `bad_quarto`) | Two packages to coordinate. The chapter's answer is a shared label such as `much_ado` |
| External store (Chubby, Bigtable, source-based filesystem) | For configs that change often or dynamically while the binary runs | — |

Owners choose between them "on a case-by-case basis."

## Anti-patterns Google warns against

- Releases that are "unique snowflakes" (non-reproducible builds).
- Accidental, unintentional changes to the release process.
- Builds that depend on the build machine's installed software or on external services (non-hermetic builds).
- Releasing directly from mainline, or merging release branches back into mainline. Both risk pulling in unrelated changes.
- Rebuilding old releases with a current compiler.
- Teams "reinventing the wheel (poorly)" instead of using tools that behave correctly by default.
- Developers who "throw the results over the fence" to release engineers.
- Treating release engineering as an afterthought, or failing to budget for it early. Retrofitting costs more.
- Not defining release processes because they can't yet be automated. They should be defined "whether or not the processes can be automated and/or enforced."

## Checklists

**Gated operations (Enforcement of Policies and Procedures):**
- [ ] Approving source code changes (config files throughout the codebase)
- [ ] Specifying actions performed during the release process
- [ ] Creating a new release
- [ ] Approving the initial integration proposal (a build request at a specific revision) and later cherry picks
- [ ] Deploying a new release
- [ ] Changing a project's build configuration

**Questions every company should answer ("It's Not Just for Googlers"):** How do you version packages? Continuous build-and-deploy or periodic builds? How often do you release? Which configuration management policies? Which release metrics?

## Short verbatim quotes

- "Running reliable services requires reliable release processes."
- "We have embraced the philosophy that frequent releases result in fewer changes between versions."
- "The build process is self-contained and must not rely on services that are external to the build environment."
- "Our goal is to fit the deployment process to the risk profile of a given service."
- "It's cheaper to put good practices and process in place early, rather than have to retrofit your system later."

## Further reading (section "More Information")

[Dic14] continuous release and change complexity; [McN13] consistency in massively parallel environments; [McN14b] "The 10 Commandments of Release Engineering"; [McN14c] distributing software (MPM).
