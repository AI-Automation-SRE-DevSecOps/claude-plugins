# Run folder and templates

Every run lives in `.sdlc-team/<run-id>/` in the project. Suggest adding `.sdlc-team/` to `.gitignore` unless the team wants the records in git.

| File | Owner | Written in |
|---|---|---|
| `brief.md` | facilitator | Intake |
| `backlog.md` | product-owner (goal, order), business-analyst (stories, AC), architect + developer (subtasks), qc-engineer (DoD) | Refinement, Planning; status updated by everyone |
| `decisions.md` | the deciding role | any step |
| `test-plan.md` | qa-engineer | Planning |
| `review.md` | architect, security-engineer | Review |
| `test-report.md` | qe-engineer (white-box), qa-engineer (black-box), sre-engineer (system & performance) | Testing |
| `ops-readiness.md` | sre-engineer | Testing |
| `qc-report.md` | qc-engineer | QC gate |
| `incident.md`, `postmortem.md` | facilitator (IC), business-analyst (timeline) | Incident workflow |
| `vuln.md` | security-engineer | CVE workflow |
| `report.md` | facilitator | End |
| `board/` | everyone (files backend only) | always |

Several roles may write to `backlog.md`. Each role edits only its own sections or fields. When two roles might write at the same time, the later one re-reads the file first.

## brief.md

```markdown
# <run-id>
Type: delivery | incident | cve      Board: mem | files      Started: <date>
## Work item
<verbatim text, ticket key or link>
## Context
Repo, stack, build/test/lint commands, CI, deploy files.
## Team
<role>: <why involved>
## Agent ids
<role>: <agent id from the Agent tool result>
```

## backlog.md

```markdown
# Backlog: <run-id>
## Goal
Product Goal: <for epics/products>
Sprint Goal: <one sentence: why this Sprint is valuable>

## Definition of Done (applies to every item)
- [ ] Code reviewed and approved by the architect; no open review issues
- [ ] Unit tests for new/changed logic; repo test suite passes: `<command>`
- [ ] Build and lint pass: `<command>`
- [ ] Every acceptance criterion verified with recorded evidence (test-report.md)
- [ ] No new high/critical findings from the dependency/secret/SAST checks that ran
- [ ] Docs/README/config samples updated where behaviour or setup changed
- [ ] Observability for new behaviour (logs/metrics/health) and rollback path noted (if it runs in production)
- [ ] No secrets, debug code or TODOs without a backlog item
<adapt to the repo; add security/SRE items when they're involved>

## Items
### ITEM-1 <title>   Priority: 1   Size: <S/M/L or points>   Status: todo|doing|review|done|carried-over
Story: As a <user>, I want <capability>, so that <benefit>.
Acceptance criteria:
- AC-1.1 Given <context>, when <action>, then <observable result>.
- AC-1.2 Given ..., when <invalid/edge case>, then ...
Non-functional: <latency/availability/security targets, if any>
Subtasks:
- [ ] T-1.1 <what> | owner: developer | depends: - | touches: <files/components> | status: todo
- [ ] T-1.2 Tests for AC-1.1..1.2 | owner: qe-engineer | depends: T-1.1 | status: todo
Notes / risks:

## Product Backlog (not in this Sprint)
- <item>: <one line, why later>
```

## decisions.md

```markdown
## D-<n> <title>   (<date>, decided by <role>)
Context: <the problem and constraints>
Options: <A / B / C with trade-offs>
Decision: <chosen option>
Consequences: <what follows, what we give up>
```

## test-plan.md

```markdown
| AC | Level (unit/integration/e2e/manual/perf/security) | Test (file or charter) | Owner | Data |
```

## test-report.md

```markdown
## White-box (qe-engineer)
Command: `<cmd>`   Result: <pass/fail counts>   Coverage of changed code: <x% or "not measured">
## Black-box (qa-engineer)
| AC | Result | Evidence (command/output/screenshot path) |
Exploratory charters: <charter → findings>
## System & performance (sre-engineer)
Environment: <local/test>   Tool: <k6/hey/...>   Load: <profile>
Results vs targets: <p50/p95/p99, error rate, resource use>
## Defects
| ID | Severity | Item/AC | Steps | Expected | Actual | Status |
```

## qc-report.md

```markdown
| Item | DoD lines failed | AC failed | Verdict (Done / Not done) | Evidence checked |
```

## incident.md

```markdown
# <run-id>   SEV<n>   Status: investigating | mitigated | resolved
Summary:
Impact: <who/what, since when, how much>
Roles: IC facilitator | Ops sre-engineer | Comms product-owner | Planning business-analyst
Current hypothesis:
Mitigation: <proposed / applied by user at hh:mm / verified by <data>>
Next update due:
## Timeline (UTC)
- hh:mm <observation | hypothesis | action | result>
```

## postmortem.md

```markdown
# Postmortem: <title>   (<date>, blameless)
Status: draft | reviewed     Authors: <roles>
Summary:
Impact: <users, duration, error budget used>
Trigger:
Root cause(s):
Detection: <how we found out; how long it took>
Resolution:
What went well / What went wrong / Where we got lucky:
Timeline (UTC):
## Action items
| Item | Type (mitigate/prevent/detect/process) | Owner role | Priority | Backlog link |
Lessons learned:
```

## vuln.md

```markdown
# <CVE-ID> in <package/component>
Advisory sources: <NVD / vendor / GHSA / OSV links>
Affected versions: <range>   Fixed in: <version>   CWE: <id>
Found in: <files, images, dependency path>
Present: yes/no   Reachable: yes/no/unknown (<how checked>)   Exposed: <internet / auth-only / internal / build-only>
Status: affected | not affected (<VEX justification>) | under investigation
CVSS: <score + vector>   EPSS: <probability, percentile, date>   KEV: yes/no
SSVC: exploitation=<> automatable=<> technical impact=<> mission/well-being=<> → <Track|Track*|Attend|Act>
Response: upgrade | patch | mitigate | compensating control | accept (<reason, expiry>)   Decided by: <role/user>
Verification: <scanner before/after, tests>
Root cause & prevention: <RV.3 findings, backlog items>
```

## report.md

```markdown
# Report: <run-id>
Outcome: <Sprint Goal met / partly / not met | incident resolved | CVE fixed/mitigated/not affected/accepted>
## Delivered (Increment)
| Item | Status | Evidence |
## Not done / carried over
## Checks that ran / didn't run
## Follow-ups (backlog)
## Lessons kept
```
