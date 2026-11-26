# Changelog

## 1.1.0
- The skill no longer knows about any memory tool. The board is always files in the run folder. If the session's own instructions (for example CLAUDE.md) describe a long-term memory or team board, roles follow those instructions.
- Roles write all run files (such as `decisions.md`) in `.sdlc-team/<run-id>/`, not the project root.
- Board posts use real clock timestamps, so they sort in the right order.

## 1.0.0
First release: the `scrum-team` skill (delivery, incident and CVE workflows), nine role subagents, and the commands `deliver`, `incident`, `cve` and `status`.
