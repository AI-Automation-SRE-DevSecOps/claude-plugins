---
description: Show where an sdlc-team run stands (step, backlog items, open board items, blockers), or list runs
argument-hint: [run-id]
---

Report the status of the sdlc-team run: $ARGUMENTS

1. Find the runs (see "Pick the run folder" in `${CLAUDE_PLUGIN_ROOT}/skills/scrum-team/references/collaboration.md`): the run folders where the session's long-term memory instructions keep team runs, if they give a place, and the folders in `.sdlc-team/`.
   If no run id is given, list the runs with "done" (has `report.md`) or "in progress", then ask which one, unless there's only one.
2. In the run folder, read `status.md`, `brief.md`, `backlog.md` and any reports.
3. Find the open questions, issues and tasks in `<run folder>/board/`.
4. Report briefly:
   - the run folder and the current workflow step
   - each item with its status
   - the open questions and issues, and who they're waiting on
   - the blockers
   - the next step

Don't change anything. To continue the run, the user can say "continue run <run-id>". Then use the `scrum-team` skill and resume from the first unfinished step.
