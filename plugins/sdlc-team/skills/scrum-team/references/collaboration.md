# How the team collaborates

Role subagents can't call each other. Every role of a run works from **one shared place**, so each one can see the current status, the backlog, the tasks and the open questions:

1. **The run folder**: the artifacts (`status.md`, brief, backlog, decisions, test reports). See `artifacts.md`. Every run file lives here, never in the project root.
2. **The board**: short messages between roles (questions, answers, decisions, issues, tasks).
3. **The facilitator** (main conversation): it spawns and continues roles, and tells each role the run folder and what to read.

## Pick the run folder (facilitator, at intake)

- **A place given by the session's long-term memory:** if the session's instructions (CLAUDE.md, AGENTS.md, Cursor rules, or text added at session start) describe a long-term memory that gives a place for a team run's files, follow those instructions to get the folder for this run id, and use it as the run folder.
- **Otherwise:** `.sdlc-team/<run-id>/` in the project. Add `.sdlc-team/` to the project's `.gitignore` if it isn't there yet, and say so in the report.

Write the run folder in `brief.md` and put it in every spawn prompt. Every role of the run uses the same run folder; never mix two places in one run. Everything else works the same wherever the folder is.

## The board

Posts are files in `<run folder>/board/`, one file per post, never edited after writing. Take the timestamp from the clock (`date -u +%Y%m%dT%H%M%S`); never make one up, because the name order is the post order.

```
board/<YYYYMMDDTHHMMSS>-<role>-<kind>-<short-title>.md
---
role: developer
kind: question        # question | issue | task | answer | decision | lesson | note | status
to: architect         # a role, or all
reply_to: <file name of the post this answers>   # answers only
ref: <file name>      # status posts only
status: closed        # status posts only
---
Body.
```

Read posts by listing the folder in name order. A question, issue or task is open until an `answer` replies to it or a `status` post closes it.

## Rules for every role

- **Write, don't edit.** Never edit or delete another role's post or file. Correct things by posting.
- **Address the owner.** Scope and value → `product-owner`. Requirements and AC → `business-analyst`. Design → `architect`. Code → `developer`. Security → `security-engineer`. Tests → `qa-engineer` / `qe-engineer`. DoD gate → `qc-engineer`. Production, performance and monitoring → `sre-engineer`.
- **Stay short.** A post is a few lines; details go in a run-folder file that the post names.
- **No secrets** in posts or artifacts: no tokens, passwords, keys or personal data.
- **Return to the facilitator** a summary of at most about 200 words: what you did, files written, posts made, open questions, blockers.

## The facilitator's loop

For each workflow step:

1. Spawn or continue (`SendMessage` to the saved agent id) the roles for the step, in parallel when they're independent. Each prompt says: run id, run folder, project folder, role, step, files to read, posts to answer (see the template in `roles.md`).
2. Read their summaries and the open board items.
3. If a post is addressed to a role that isn't active, bring that role in or continue it with "answer the open board items addressed to you".
4. Repeat until the step has no open questions or issues addressed to the step's roles, or until the step's exit criteria are met.
5. Update `status.md` (see `artifacts.md`), then move on.
6. A blocker nobody on the team can resolve goes to the user.

### Optional: real agent teams

If the user has turned on Claude Code agent teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, interactive sessions only), roles can be spawned as named teammates that message each other directly and share a task list. Use the same role agent types, the same run folder and the same artifacts. Keep the board too, as the written record. Teammates can't spawn teammates, so the facilitator stays the only one who brings roles in.

## After the run: keep what was learned

In the retrospective, each role adds at most two lessons, and the team's lasting decisions are already on the board. List them in `report.md`.
Saving them anywhere else (a long-term memory, notes, docs) is up to the session's own instructions, not this skill: follow those if there are any.

Keep only lessons that help the next run. Don't keep: one-off details of this task, anything already in the code or git history, guesses, secrets, or folder paths of this working copy.
