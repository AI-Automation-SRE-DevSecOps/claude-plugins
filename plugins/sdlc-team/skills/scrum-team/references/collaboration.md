# How the team collaborates

Role subagents can't call each other. They share three things:

1. **The run folder** `.sdlc-team/<run-id>/`: the artifacts (brief, backlog, decisions, test reports). See `artifacts.md`. Every run file lives here, never in the project root.
2. **The board**: short messages between roles (questions, answers, decisions, issues), as files in the run folder.
3. **The facilitator** (main conversation): it spawns and continues roles, and tells each role which board posts and files to read.

## Board

Posts are files in `.sdlc-team/<run-id>/board/`, one file per post, never edited after writing. Take the timestamp from the clock (`date -u +%Y%m%dT%H%M%S`); never make one up, because the name order is the post order.

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

## Long-term memory

This skill doesn't depend on any memory tool. If the session's instructions (for example CLAUDE.md or project rules) describe a long-term memory or a shared team board, the facilitator and every role follow those instructions as well: check memory before a step, and save lessons and decisions the way they say. Pass the run id and each role's name in every spawn prompt so those instructions can use them. The board files in the run folder stay the record of the run either way.

## Rules for every role

- **Write, don't edit.** Never edit or delete another role's post or file. Correct things by posting.
- **Address the owner.** Scope and value → `product-owner`. Requirements and AC → `business-analyst`. Design → `architect`. Code → `developer`. Security → `security-engineer`. Tests → `qa-engineer` / `qe-engineer`. DoD gate → `qc-engineer`. Production, performance and monitoring → `sre-engineer`.
- **Stay short.** A post is a few lines; details go in a run-folder file that the post links to.
- **No secrets** in posts, artifacts or memory: no tokens, passwords, keys or personal data.
- **Return to the facilitator** a summary of at most about 200 words: what you did, files written, posts made, open questions, blockers.

## The facilitator's loop

For each workflow step:

1. Spawn or continue (`SendMessage` to the saved agent id) the roles for the step, in parallel when they're independent. Each prompt says: run id, run folder, role, step, files to read, posts to answer.
2. Read their summaries and the open board items.
3. If a post is addressed to a role that isn't active, bring that role in or continue it with "answer the open board items addressed to you".
4. Repeat until the step has no open questions or issues addressed to the step's roles, or until the step's exit criteria are met. Then move on.
5. A blocker nobody on the team can resolve goes to the user.

### Optional: real agent teams

If the user has turned on Claude Code agent teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, interactive sessions only), roles can be spawned as named teammates that message each other directly and share a task list. Use the same role agent types and the same artifacts. Keep the board files too, as the written record. Teammates can't spawn teammates, so the facilitator stays the only one who brings roles in.

## After the run: keep what was learned

In the retrospective, each role posts at most two `lesson` items, and the team's lasting `decision` items are already on the board. List them in `report.md`. If the session has long-term memory instructions, save them the way those instructions say.

Don't save: one-off details of this task, anything already in the code or git history, guesses, or secrets.
