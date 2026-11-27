# How the team collaborates

Role subagents can't call each other. Every role of a run works from **one shared place**, so each one can see the current status, the backlog, the tasks and the open questions:

1. **The run folder**: the artifacts (`status.md`, brief, backlog, decisions, test reports). See `artifacts.md`. Every run file lives here, never in the project root.
2. **The board**: short messages between roles (questions, answers, decisions, issues, tasks).
3. **The facilitator** (main conversation): it spawns and continues roles, and tells each role the store, the run folder, the board commands, and what to read.

## Pick the store (facilitator, at intake)

**Memory store** when the long-term memory tool agent-memory is enabled. Both must be true:
- the session's instructions mention it (an "agent-memory" section in CLAUDE.md, AGENTS.md or Cursor rules), or `memory_*` / `board_*` tools are available;
- `mem board path --run <run-id>` (or the `board_path` tool) works. It prints the run folder and creates it.

If it answers "no project here", memory found no project for this folder (no git repo, and no project file such as `package.json` or `pyproject.toml`). Ask the user which project the run is for, and use `--dir <that project folder>` (`project_dir`) on every `mem` call of the run.

**File store** otherwise: the run folder is `.sdlc-team/<run-id>/` in the project. Add `.sdlc-team/` to the project's `.gitignore` if it isn't there yet, and say so in the report.

Write the choice in `brief.md` (`Store: memory` or `Store: files`, plus the run folder) and put it in every spawn prompt. Never mix the two in one run.

| | Memory store | File store |
|---|---|---|
| Run folder | the path printed by `mem board path --run <run-id>` (inside the project's memory, not in the repo) | `.sdlc-team/<run-id>/` |
| Board | `mem board` commands (below) | one file per post in `<run folder>/board/` (below) |
| List runs (`/sdlc-team:status`, resume) | `mem board runs` | the folders in `.sdlc-team/` |
| Lessons at the end | `lesson` / `decision` posts, then `mem board promote --run <run-id>` | listed in `report.md` |

## Board, memory store

Use the `mem` command (or the `board_*` tools with the same fields). Add `--dir <project folder>` if the session isn't in the project folder.

```bash
# read what's waiting for you, before each step
mem board read --run <run-id> --for <your-role> --status open
# ask, report, decide, hand out work
mem board post --run <run-id> --role <your-role> --kind question --to <role> --title "<one line>" [--body "<a few lines>"]
#   kinds: question | issue | task | decision | lesson | note
# answer a post (its id is the first word of the read output)
mem board reply <post-id> --run <run-id> --role <your-role> --to <role> --title "<one line>"
# close a question, issue or task that is resolved
mem board close <post-id> --run <run-id> --role <your-role> [--body "<why>"]
# a whole conversation
mem board thread <post-id> --run <run-id>
```

A question, issue or task is open until someone replies to it or closes it.

## Board, file store

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
- **No secrets** in posts, artifacts or memory: no tokens, passwords, keys or personal data.
- **Return to the facilitator** a summary of at most about 200 words: what you did, files written, posts made, open questions, blockers.

## The facilitator's loop

For each workflow step:

1. Spawn or continue (`SendMessage` to the saved agent id) the roles for the step, in parallel when they're independent. Each prompt says: run id, store, run folder, project folder, role, step, files to read, posts to answer (see the template in `roles.md`).
2. Read their summaries and the open board items.
3. If a post is addressed to a role that isn't active, bring that role in or continue it with "answer the open board items addressed to you".
4. Repeat until the step has no open questions or issues addressed to the step's roles, or until the step's exit criteria are met.
5. Update `status.md` (see `artifacts.md`), then move on.
6. A blocker nobody on the team can resolve goes to the user.

### Optional: real agent teams

If the user has turned on Claude Code agent teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, interactive sessions only), roles can be spawned as named teammates that message each other directly and share a task list. Use the same role agent types, the same store and the same artifacts. Keep the board too, as the written record. Teammates can't spawn teammates, so the facilitator stays the only one who brings roles in.

## After the run: keep what was learned

In the retrospective, each role posts at most two `lesson` items on the board, and the team's lasting `decision` items are already there. List them in `report.md`.
- **Memory store:** the facilitator runs `mem board promote --run <run-id>`, which turns the lesson and decision posts into notes of this project.
- **File store:** if the session's instructions describe another long-term memory, save them the way those say.

Keep only lessons that help the next run. Don't keep: one-off details of this task, anything already in the code or git history, guesses, secrets, or folder paths of this working copy.
