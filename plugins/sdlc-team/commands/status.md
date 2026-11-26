---
description: Show where an sdlc-team run stands (step, backlog items, open board items, blockers), or list runs
argument-hint: [run-id]
---

Report the status of the sdlc-team run: $ARGUMENTS

1. If no run id is given, list the folders in `.sdlc-team/` with their type and the first line of `report.md` (or "in progress"), then ask which one, unless there's only one.
2. Read `brief.md`, `backlog.md` and any reports in `.sdlc-team/<run-id>/`.
3. Read the board: `mem board read --run <run-id> --status open` if `brief.md` says `Board: mem`, otherwise the files in `board/`.
4. Report briefly:
   - the current workflow step
   - each item with its status
   - the open questions and issues, and who they're waiting on
   - the blockers
   - the next step

Don't change anything. To continue the run, the user can say "continue run <run-id>". Then use the `scrum-team` skill and resume from the first unfinished step.
