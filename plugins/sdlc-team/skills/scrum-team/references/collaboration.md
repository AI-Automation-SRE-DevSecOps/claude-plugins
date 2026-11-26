# How the team collaborates

Role subagents can't call each other. They share three things:

1. **The run folder** `.sdlc-team/<run-id>/`: the artifacts (brief, backlog, decisions, test reports). See `artifacts.md`.
2. **The board**: short messages between roles (questions, answers, decisions, issues).
3. **The facilitator** (main conversation): it spawns and continues roles, and tells each role which board posts and files to read.

## Board: pick the backend once per run

At run setup, run `command -v mem`. Record the backend in `brief.md` (`board: mem` or `board: files`) and give it to every role in its spawn prompt.

### Backend `mem` (agent-memory installed)

The board lives in long-term memory and the team's lessons can be kept after the run.

| Action | Command |
|---|---|
| Read posts for me | `mem board read --run <run-id> --for <role> --json` |
| Read only open items | `mem board read --run <run-id> --status open` |
| Read a thread | `mem board thread <post-id> --json` |
| Ask / raise | `mem board post --run <run-id> --role <role> --kind question\|issue\|task --to <role\|all> --title "..." --body "..."` |
| Answer | `mem board reply <post-id> --run <run-id> --role <role> --body "..."` |
| Record a decision | `mem board post --run <run-id> --role <role> --kind decision --to all --title "..." --body "<reason first line>\n<details>"` |
| Record a lesson | `mem board post --run <run-id> --role <role> --kind lesson --to all --title "..." --body "..."` |
| Close an item | `mem board close <post-id> --run <run-id> --role <role> --body "why"` |

Before starting a step, a role should also check memory for prior knowledge: `mem recall "<topic> <role>"`. When a recalled note helped, `mem feedback <id> success`.

### Backend `files` (no agent-memory)

Posts are files in `.sdlc-team/<run-id>/board/`, one file per post, never edited after writing:

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
- **Stay short.** A post is a few lines; details go in a run-folder file that the post links to.
- **No secrets** in posts, artifacts or memory: no tokens, passwords, keys or personal data.
- **Return to the facilitator** a summary of at most about 200 words: what you did, files written, posts made, open questions, blockers.

## The facilitator's loop

For each workflow step:

1. Spawn or continue (`SendMessage` to the saved agent id) the roles for the step, in parallel when they're independent. Each prompt says: run id, run folder, board backend, role, step, files to read, posts to answer.
2. Read their summaries and the open board items.
3. If a post is addressed to a role that isn't active, bring that role in or continue it with "answer the open board items addressed to you".
4. Repeat until the step has no open questions or issues addressed to the step's roles, or until the step's exit criteria are met. Then move on.
5. A blocker nobody on the team can resolve goes to the user.

### Optional: real agent teams

If the user has turned on Claude Code agent teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, interactive sessions only), roles can be spawned as named teammates that message each other directly and share a task list. Use the same role agent types and the same artifacts. Keep the board too, as the written record. Teammates can't spawn teammates, so the facilitator stays the only one who brings roles in.

## After the run: keep what was learned

In the retrospective, each role posts at most two `lesson` items and the team's lasting `decision` items are already on the board. Then:

- Backend `mem`: run `mem board promote --run <run-id>` to copy decisions and lessons into project memory, then `mem board archive --run <run-id>`. Lessons about the user or about all projects can be saved with `mem save --scope global`.
- Backend `files`: list the lessons and decisions in `report.md` so they can be added to the project's notes by hand.

Don't save: one-off details of this task, anything already in the code or git history, guesses, or secrets.
