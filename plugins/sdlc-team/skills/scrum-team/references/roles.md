# Who to involve, and when

Each role is a subagent in `agents/`. Its own file holds its knowledge and rules. This table is for the facilitator.

| Step ↓ / Role → | PO | BA | Architect | Dev | Security | QA | QE | QC | SRE |
|---|---|---|---|---|---|---|---|---|---|
| Refinement | ● | ● | ● | sizing | if risky | AC testability | – | – | if production / NFRs |
| Sprint Planning | ● Sprint Goal | – | ● subtasks | ● subtasks | DoD items | ● test plan | – | ● DoD | DoD items |
| Implementation | answers | answers | answers | ● | – | – | ● test code | – | – |
| Review & white-box | – | – | ● code review | fixes | ● scans, secure review | – | ● | – | – |
| Black-box & system | – | – | – | fixes | – | ● | automation | – | ● perf, observability |
| QC gate | – | – | – | – | – | – | – | ● | – |
| Sprint Review | ● accepts | – | – | – | follow-ups | evidence | – | – | evidence, follow-ups |
| Incident | Comms lead | Planning lead | specialist | specialist | if security | – | – | fix gate | Ops lead |
| CVE | priority, risk acceptance | – | breaking changes | ● fix | ● lead | regression | regression | ● | exposure, rollout |

● = owns or leads the step.

## Involve a role when

- **Security engineer**: authentication, authorization, sessions, user input reaching queries, commands or templates, file uploads, secrets, cryptography, PII, new dependencies, network exposure, IaC or permissions, any CVE.
- **SRE**: the change runs in production, has users or load, adds a service, queue, database, cron or external dependency, changes timeouts, retries or capacity, or needs monitoring or rollout planning. Also for any incident.
- **Architect**: more than one component changes, a new interface or data model, a technology choice, or the change is hard to undo.
- **BA**: the input is vague, has several user types, business rules, or is an epic.
- **QA, QE, QC**: any change to behaviour. For a pure refactor, QE and QC are enough, with "behaviour unchanged" as the AC.

## QA vs QE vs QC

| | QA, quality assurance | QE, quality engineering | QC, quality control |
|---|---|---|---|
| Question | Are we building the right thing, and can we prove it? | Is the code tested well from the inside? | Does this item meet its AC and the DoD, yes or no? |
| When | Refinement → Planning → black-box testing | Implementation → white-box testing | After testing, before the PO accepts |
| Looks at | Requirements, behaviour from the outside | Code paths, test code, coverage, automation, test infrastructure | Evidence, plus an independent re-run |
| Output | Testable AC, test plan, acceptance and exploratory results | Unit, integration and performance test code; coverage | Done / Not done verdict, defect reports |

## Spawn prompt template

```
Run: <run-id>   Your role: <role>
Store: memory | files   Run folder: <absolute path>   Project folder: <repo path>
Board: <memory: mem board read --run <run-id> --for <role> --status open; post/reply/close as in collaboration.md, with --dir <project folder> if needed>
       <files: one file per post in <run folder>/board/, format in collaboration.md>
Step: <step name> (<workflow file>)
Read: status.md, brief.md, backlog.md<, other files> in the run folder. Board items addressed to you: <ids or "read the board">.
Do: <what this role must produce in this step>
Write: <file/section> inside the run folder. Post questions/decisions/issues on the board.
Return: ≤200 words — done, files written, posts, open questions, blockers.
```
