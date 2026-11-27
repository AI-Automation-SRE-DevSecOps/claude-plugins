---
description: Show where an sdlc-team run stands (step, backlog items, open board items, blockers), or list runs
argument-hint: [run-id]
---

Report the status of the sdlc-team run: $ARGUMENTS

1. Find the runs in both stores (see "Pick the store" in `${CLAUDE_PLUGIN_ROOT}/skills/scrum-team/references/collaboration.md`):
   - memory store: `mem board runs` (if agent-memory is enabled; add `--dir <project folder>` if needed), which prints each run's folder;
   - file store: the folders in `.sdlc-team/`.
   If no run id is given, list the runs with their store and "done" or "in progress", then ask which one, unless there's only one.
2. In the run folder, read `status.md`, `brief.md`, `backlog.md` and any reports.
3. Find the open questions, issues and tasks on the board: `mem board read --run <run-id> --status open` (memory store), or the files in `<run folder>/board/` (file store).
4. Report briefly:
   - the store and the current workflow step
   - each item with its status
   - the open questions and issues, and who they're waiting on
   - the blockers
   - the next step

Don't change anything. To continue the run, the user can say "continue run <run-id>". Then use the `scrum-team` skill and resume from the first unfinished step.
