# Delivery workflow (Scrum)

For a task, user story, epic, feature, bug or ticket. The shape follows the Scrum Guide: one Sprint per run, with a Sprint Goal, a Sprint Backlog and an Increment that meets the Definition of Done. Large inputs (an epic, "build a chatbot") become a Product Backlog. The first Sprint takes the top items, and the rest stay in the backlog for later runs.

```
0 Intake → 1 Refinement → 2 Sprint Planning → [user approves] → 3 Implementation
  → 4 Review & white-box testing → 5 Black-box, system & performance testing
  → 6 QC gate (DoD) → 7 Sprint Review (PO accepts) → 8 Retrospective & report
            ↑______________ rejected items go back with a defect report ______|
```

Each step lists the roles, what they produce, and its **exit criteria**. Don't leave a step until they're met.

---

## 0. Intake (facilitator)

- Pick the store and get the run folder (see "Pick the store" in `collaboration.md`).
- Write `brief.md`: the work item verbatim, its source (ticket key, user), the store and run folder, the repo and stack you can see, and who you're bringing in (see `roles.md`). Write the first `status.md`.
- Scan the repository just enough to describe it: languages, build and test commands, CI, deployment files. Write it under `## Context` in `brief.md`.
- **Exit:** `brief.md` exists and the team is picked.

## 1. Refinement

Roles: **product-owner**, **business-analyst**, **architect**. Add **security-engineer**, **sre-engineer** or **qa-engineer** when the item touches their area. Run them in parallel. The developer joins for sizing.

- PO: the **Product Goal** (for large items) and the value of this item. Who is it for, and what outcome does it achieve? Writes `## Goal` in `backlog.md`.
- BA: the user stories (`As a <user>, I want <capability>, so that <benefit>`) and **acceptance criteria** in Given/When/Then, including unhappy paths, edge cases, validation, permissions and empty states. Also lists assumptions and questions for the PO. Checks each story against INVEST (independent, negotiable, valuable, estimable, small, testable).
- Architect: the design sketch (components, data, interfaces, dependencies), options with trade-offs when there's a real choice, and technical risks. Records significant choices as ADRs in `decisions.md`.
- Security (if involved): a threat model of the new surface (STRIDE-style: spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege) and security AC.
- SRE (if involved): non-functional requirements: an SLO or latency/availability targets, expected load, data retention, observability, rollout and rollback needs.
- QA: does each AC have an observable, testable outcome? Rewrites or flags vague ones ("fast", "user-friendly").
- Developer: sizes each item (story points or S/M/L) and flags items too big for one Sprint. The Scrum Guide says the developers who do the work size it.
- Questions go on the board, and the PO answers scope questions. Questions only the user can answer are collected and asked **once**, in one message.
- **Exit:** every candidate item has a story, testable AC, a size, and no open questions blocking it.

## 2. Sprint Planning

Roles: **product-owner**, **architect**, **developer**, **qa-engineer**, **qc-engineer** (plus security and SRE if involved).

- PO orders the backlog and proposes the **Sprint Goal**: one sentence saying why this Sprint is valuable.
- Team agrees the **Definition of Done** for the run. Start from the default in `artifacts.md` and adapt it to the repo, for example the repo's real test, lint and build commands. Security and SRE add their items when involved. QC owns the final wording.
- Architect and developer split each selected item into **subtasks** of one day or less, each with an owner role, dependencies, and the files or components it touches. Every subtask that changes behaviour includes its tests.
- QA writes the **test plan** in `test-plan.md`: which AC are covered by which test level (unit, integration, end-to-end, manual/exploratory, performance, security), and the test data.
- Write it all in `backlog.md` using the template in `artifacts.md`.
- **Checkpoint:** show the user the Sprint Goal, the items with AC, the DoD, the subtasks and the test plan summary. Ask them to approve or change it. Don't start implementing before they approve, unless they said to run without stopping.
- **Exit:** the user approves (or waived approval), and the backlog has items with AC, a DoD, subtasks with owners, and a test plan.

## 3. Implementation

Roles: **developer** (one per independent subtask when they don't touch the same files; otherwise one developer, sequentially). **qe-engineer** writes test code in parallel with the developer where the interfaces are clear (tests first is fine).

- Developers work subtask by subtask: read the code, implement, write or update unit tests, run the repo's build, lint and tests, then update the subtask status in `backlog.md` (`todo → doing → review`).
- Parallel developers on overlapping files conflict. If two subtasks touch the same files, run them one after another. For truly independent work in a git repo, you may spawn developers with `isolation: "worktree"` and merge their branches afterwards.
- Design questions → architect. Requirement questions → BA/PO. A developer must not change scope or AC on its own.
- **Exit:** every subtask is in `review`, the build passes, and the unit tests pass (or failures are explained on the board).

## 4. Code review and white-box testing

Roles: **architect** (code review), **security-engineer** (secure code review and scans), **qe-engineer** (white-box tests). Run in parallel.

- Architect reviews the diff the way Google's code review guide says: design first, then functionality, complexity, tests, naming, comments, style, consistency, documentation. Approve when the change clearly improves code health, even if it isn't perfect. Prefix optional comments with `Nit:`. Findings go in `review.md` with file:line, and real issues become board `issue` posts to the developer.
- Security runs what the repo can run (dependency audit such as `npm audit`, `pip-audit`, `govulncheck`, `osv-scanner`, `trivy`; SAST such as `semgrep` if present; a secret scan). It also reviews input validation, authn/authz, injection, secrets handling, logging of sensitive data, and the security AC. It reports exactly which tools ran and which weren't available.
- QE: unit and integration tests against the code paths (branches, error handling, boundaries), coverage of the changed code, and tests that fail when the code is broken (mutation-style spot checks). Results go in `test-report.md` under "White-box".
- Developer fixes the issues, and reviewers re-check only what changed.
- **Exit:** no open `issue` posts from review, security or QE, unless the PO accepted one as a known limitation in `decisions.md`.

## 5. Black-box, system and performance testing

Roles: **qa-engineer**, **sre-engineer** (plus **qe-engineer** for automation). Run in parallel.

- QA tests the running behaviour against every AC without reading the implementation: acceptance tests per Given/When/Then, exploratory testing (charters such as boundaries, invalid input, interruptions, concurrency, permissions), and regression of nearby features. Each AC is marked pass/fail with evidence (command, output, screenshot path) in `test-report.md` under "Black-box".
- SRE runs system-level checks the environment allows: it starts the service locally or in a test environment, runs a load or latency test against the non-functional targets (`k6`, `hey`, `wrk`, `ab`, `locust`, or a small script), checks resource use, and checks that logs, metrics, health checks and alerts exist for the new behaviour. It writes the **operational readiness** notes: SLIs to watch, dashboards, alerts, rollout plan (canary or staged), rollback steps, and runbook entries. Results go in `test-report.md` under "System & performance" and `ops-readiness.md`.
- Don't run load tests against shared or production environments without the user's approval.
- Failures become `issue` posts to the developer, with steps to reproduce, expected and actual results, and severity. Fixes go through step 3 → 4 → 5 again, only for the affected items.
- **Exit:** every AC has a recorded result, and failures are fixed or accepted by the PO.

## 6. QC gate (Definition of Done)

Role: **qc-engineer**. It is independent: it didn't build or test the item.

- For each item, QC checks every DoD line and every AC against the evidence in `test-report.md`, `review.md` and the code. Where evidence is missing it re-runs the repo's tests itself. It also re-runs a regression suite.
- Verdict per item: **Done** or **Not done** with the failed DoD/AC lines, in `qc-report.md`.
- Not-done items go back to step 3 with the defect report. After 3 failed rounds on the same item, stop and ask the user (SKILL.md rule 8).
- **Exit:** every Sprint item is Done, or has been explicitly carried back to the Product Backlog.

## 7. Sprint Review

Role: **product-owner** (with QA and SRE to present evidence).

- The PO inspects the Increment against the Sprint Goal and each item's AC, and accepts or rejects each item with a reason. Only Done items can be accepted.
- The PO updates the Product Backlog: new items found during the Sprint, carried-over items, and follow-ups from security and SRE (for example "add alert X before launch").
- Present the outcome to the user: what is in the Increment, the evidence, what was not done and why, and the proposed next Sprint. Pushing, PRs, releases and deployment need the user's go-ahead (SKILL.md rule 7).
- **Exit:** every item is accepted or carried over, and the user has seen the review.

## 8. Retrospective and report

Roles: all roles that took part. Continue each with one short prompt.

- Each role posts at most two `lesson` items on the board: what went well, what to change. Facts about the repo that will help next time count, for example "integration tests need `docker compose up db` first".
- The facilitator writes `report.md` using the template in `artifacts.md`, sets `status.md` to done, and keeps the lessons as described in `collaboration.md` (memory store: `mem board promote --run <run-id>`).
- **Exit:** `report.md` is written and lessons are kept.

---

## Scaling

| Size | Steps that shrink |
|---|---|
| Typo, config tweak, one-line bug | Intake → developer → QC (DoD = tests pass, reviewed) → PO accepts. One checkpoint before changing code. |
| One story | All steps; BA and architect may be one combined spawn each; SRE only if it runs in production |
| Epic / new product (e.g. "create a chatbot") | Step 1 produces a full Product Backlog (epics → stories) and a release plan. Steps 2–8 run for the first Sprint's items only. The report lists the next Sprints. |
| Bug | Step 1 adds a reproduction by QA or the developer, and the AC include "the reproduction no longer fails" plus a regression test |
