# Changelog

## 1.2.0
- One shared place per run. With agent-memory enabled, the run folder and board live in that project's memory (`mem board path`, `mem board` commands), and the retro's lessons become project notes (`mem board promote`). Without it, the run uses `.sdlc-team/<run-id>/` and adds it to `.gitignore`.
- New `status.md` in every run: current step, item status, who's waiting on what, blockers, next step. Updated after each step; every role reads it first.
- Spawn prompts carry the store, run folder, project folder and board commands, so every role uses the same place.
- Outside a git repo, the team asks which project the run is for.
- `/sdlc-team:status` finds runs in both stores.

## 1.1.0
- The skill no longer knows about any memory tool. The board is always files in the run folder. If the session's own instructions (for example CLAUDE.md) describe a long-term memory or team board, roles follow those instructions.
- Roles write all run files (such as `decisions.md`) in `.sdlc-team/<run-id>/`, not the project root.
- Board posts use real clock timestamps, so they sort in the right order.

## 1.0.0
First release: the `scrum-team` skill (delivery, incident and CVE workflows), nine role subagents, and the commands `deliver`, `incident`, `cve` and `status`.
