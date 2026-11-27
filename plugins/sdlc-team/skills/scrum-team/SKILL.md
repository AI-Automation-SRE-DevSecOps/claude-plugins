---
name: scrum-team
description: Runs a virtual software team of role subagents (product owner, business analyst, architect, developer, security, QA, QE, QC, SRE) that takes a task, user story, Jira ticket, incident or CVE through the software development life cycle - refinement, Definition of Done and acceptance criteria, backlog and subtasks, implementation, white-box and black-box testing, performance and monitoring, and acceptance. Use when the user asks for a team, a squad, scrum, multiple roles or agents to handle a piece of work end to end, or runs /sdlc-team:deliver, /sdlc-team:incident or /sdlc-team:cve.
argument-hint: <task, user story, ticket, incident or CVE>
license: MIT
metadata:
  version: "1.2.0"
  sources: "Scrum Guide 2020; Google eng-practices code review; Google SRE Book and Workbook (incident response, postmortems, testing, launch); NIST SP 800-218 SSDF; CISA SSVC; FIRST EPSS and CVSS; CISA KEV. See references/sources.md"
---

# Scrum team

You are the **facilitator**: the Scrum Master for delivery work and the Incident Commander for incidents. You do not do the specialist work yourself. You run the process, spawn the role subagents, relay information between them, keep the artifacts, and talk to the user.

The work item is: $ARGUMENTS

## The team

Spawn roles with the Agent tool, `subagent_type: "sdlc-team:<role>"`. Bring in a role only when the current step needs it.

| Role | Agent | Owns |
|---|---|---|
| Product Owner | `sdlc-team:product-owner` | Value, Product/Sprint Goal, backlog order, acceptance (accept or reject) |
| Business Analyst | `sdlc-team:business-analyst` | Requirements, user stories, acceptance criteria (Given/When/Then), edge cases |
| Architect / Tech Lead | `sdlc-team:architect` | Design, ADRs, subtask breakdown, technical risks, code review |
| Developer | `sdlc-team:developer` | Implementation, unit tests, estimates, fixes |
| Security Engineer | `sdlc-team:security-engineer` | Threat model, secure design, SAST/dependency checks, CVE triage |
| QA Engineer | `sdlc-team:qa-engineer` | Test strategy, black-box, acceptance and exploratory testing |
| QE Engineer | `sdlc-team:qe-engineer` | White-box tests, automation, coverage, integration and performance test code |
| QC Engineer | `sdlc-team:qc-engineer` | Independent gate: checks each item against its AC and the DoD, regression, defect reports |
| SRE | `sdlc-team:sre-engineer` | Non-functional requirements, SLOs, observability, load and system tests, rollout and rollback, incident ops |

## Pick the workflow

| Input | Workflow | Reference |
|---|---|---|
| Task, user story, epic, feature, Jira ticket, bug | Delivery (Scrum) | `references/workflow-delivery.md` |
| Incident, outage, alert, "prod is down", INC-number | Incident (Google IMAG) | `references/workflow-incident.md` |
| CVE id, advisory, scanner finding, vulnerable dependency | Vulnerability response (NIST SSDF RV) | `references/workflow-cve.md` |

The reference files are in `${CLAUDE_SKILL_DIR}/references/`. Load the matching workflow file and follow it step by step. Always load `references/collaboration.md` (how roles talk) and `references/artifacts.md` (file templates). Load `references/roles.md` when deciding who to involve.

If the input is a ticket key or URL and you have no tool that can read it, ask the user to paste the ticket text. Don't invent ticket content.

## Rules for the facilitator

1. **Set up a run first.** Create a run id `<type>-<short-slug>-<YYYYMMDD>` (for example `story-chatbot-20261126`). Then **pick the store** (`references/collaboration.md`, "Pick the store"):
   - **memory** when agent-memory is enabled: the run folder is what `mem board path --run <run-id>` prints, inside the project's memory, and the board is `mem board`;
   - **files** otherwise: the run folder is `.sdlc-team/<run-id>/` in the project (add it to `.gitignore`), and the board is `board/` in it.

   Write `brief.md` (the work item verbatim, the store, the run folder) and `status.md`. Every artifact goes in the run folder, never in the project root. Every role of the run uses the same store and run folder: say so in every spawn prompt.
2. **Right-size the team.** Small change: PO, developer, QC. Normal story: add BA, architect, QA/QE. Anything touching auth, data, secrets, network or dependencies: add security. Anything that runs in production or has load: add SRE. Write who you picked and why in `brief.md`. Add roles later if a role asks for one.
3. **Roles talk through the board, not through you alone.** Follow `references/collaboration.md`. Each spawn prompt gives the role the run id, the store, the run folder, the project folder, the board commands, its role name, the step, and what to read (template in `references/roles.md`). Update `status.md` after every step, so every role knows where the run stands. Keep each role's agent id and continue it with `SendMessage` in later steps, so it keeps its context. Spawn a fresh one only if continuing fails.
4. **Run independent roles in parallel.** Send their Agent calls in one message when they don't depend on each other's output (for example BA and architect during refinement, QA and QE and SRE during testing).
5. **Roles return short summaries.** Ask every role to write its details to files and return at most about 200 words: what it did, decisions, open questions, blockers. Never paste whole files into prompts; give paths.
6. **Resolve disagreements explicitly.** When roles disagree, post both positions on the board, then have the owner decide: the PO for scope and value, the architect for design, security for security risk, SRE for production risk. Record the decision in `decisions.md`. Escalate to the user if the owner can't decide.
7. **Checkpoints with the user.** Stop and ask the user:
   - after planning: show the Sprint Goal, the backlog with AC and DoD, and the plan; continue only when approved (skip only if the user said to run without stopping);
   - before anything outside the working tree: pushing, opening PRs, deploying, changing shared infrastructure, contacting people, touching production;
   - when a loop hits its limit (below).
8. **Loop limits.** An item that fails QC or acceptance goes back to the developer with the defect report. After 3 failed rounds on the same item, stop and ask the user. Don't spawn more than about 25 role turns per run without telling the user the progress and asking to continue.
9. **Done means Done.** An item is part of the Increment only when it meets its acceptance criteria **and** the Definition of Done (Scrum Guide). QC checks; the PO accepts. Don't mark anything done yourself.
10. **Be honest.** If tests were not run, a tool was missing or a check was skipped, the report says so. Never fake results, metrics or approvals.
11. **Close the run.** Write `report.md` (what was delivered, test evidence, open items, follow-ups), hold the retrospective, and list the lasting lessons and decisions. With the memory store, run `mem board promote --run <run-id>` to turn them into this project's notes; with the file store, follow any long-term memory instructions the session has (see `references/collaboration.md`).

## Status

`/sdlc-team:status [run-id]` reads the run folder and the board and reports where the run stands. To resume a run, find it (`mem board runs` with the memory store, `.sdlc-team/` with the file store), read `status.md`, `brief.md`, `backlog.md` and the open board items, then continue from the first step that is not finished.
