# claude-plugins

A Claude Code plugin marketplace for SRE, DevSecOps and software delivery work. Every plugin here has been reviewed before publishing.

## Install

No need to clone anything. In Claude Code, run:

```
/plugin marketplace add AI-Automation-SRE-DevSecOps/claude-plugins
/plugin install sre-engineering@sre-devsecops
/plugin install sdlc-team@sre-devsecops
```

Then restart Claude Code so the new skills and commands are loaded.

You can also run `/plugin` with no arguments to browse and manage plugins from a menu.

### Update

```
/plugin marketplace update sre-devsecops
```

### Uninstall

```
/plugin uninstall sre-engineering@sre-devsecops
```

## Plugins

| Plugin | What it does |
|---|---|
| [`sre-engineering`](plugins/sre-engineering/) | Google SRE practice from the SRE Book and SRE Workbook: SLOs and error budget policy, monitoring and burn-rate alerting, on-call, incidents, postmortems, toil, releases, overload, capacity and launch reviews |
| [`sdlc-team`](plugins/sdlc-team/) | A virtual software team of subagents (PO, BA, architect, developer, security, QA, QE, QC, SRE) that takes a task, story, incident or CVE through the whole development life cycle, Scrum-style |

### `sre-engineering`

**Skill:** `sre-engineer`. Claude loads it automatically for SRE questions: SLOs, alerting, on-call, incidents, postmortems, toil, rollouts, overload and capacity. You can also call it directly with `/sre-engineering:sre-engineer`. Its reference files are summaries of every chapter of [*Site Reliability Engineering*](https://sre.google/sre-book/table-of-contents/) and [*The Site Reliability Workbook*](https://sre.google/workbook/table-of-contents/). `references/sources.md` maps each chapter to a file.

**Commands:**

| Command | What it does |
|---|---|
| `/sre-engineering:define-slo <service>` | Defines SLIs, SLOs on a 4-week window, the error budget, multiwindow burn-rate alerts, the error budget policy and an SLO document |
| `/sre-engineering:design-monitoring <service>` | Designs monitoring and alerting: golden signals, metrics with purpose, burn-rate alerts, an alert review table, playbooks, dashboards, tests and config |
| `/sre-engineering:review-alerts [file or dir]` | Reviews alert rules against Google's five questions and burn-rate practice, and recommends keep / change / downgrade / delete / automate |
| `/sre-engineering:write-postmortem <incident>` | Drafts or reviews a blameless postmortem that separates trigger from root cause and has typed, owned action items |
| `/sre-engineering:production-readiness-review <service>` | Runs a PRR or launch review against Google's launch checklist, and ranks findings by error budget impact |

Examples:

```
/sre-engineering:define-slo payments API
/sre-engineering:design-monitoring checkout API on EKS behind an ALB
/sre-engineering:review-alerts terraform/aws/modules/monitoring
/sre-engineering:write-postmortem incidents/2026-11-20-checkout-outage.md
/sre-engineering:production-readiness-review services/checkout
```

See [CHANGELOG.md](plugins/sre-engineering/CHANGELOG.md) for everything corrected in 2.0.0. The per-chapter source notes are in [docs/sre-notes/](docs/sre-notes/).

### `sdlc-team`

You give it a work item. The main conversation acts as **Scrum Master** (or **Incident Commander** for incidents) and brings in a team of role subagents as each step needs them. The roles discuss on a shared board, agree on the Definition of Done and acceptance criteria, split the work into subtasks, build it, test it and accept it.

**Commands:**

| Command | What it does |
|---|---|
| `/sdlc-team:deliver <task / story / ticket>` | Scrum delivery: refinement → Sprint Goal, AC, DoD, backlog, subtasks → **your approval** → implement → code review and white-box tests → black-box, system and performance tests → QC gate → PO acceptance → retro and report |
| `/sdlc-team:incident <what is happening>` | Google SRE incident response: declare → mitigate first (**you approve and run every production action**) → root cause → fix → blameless postmortem |
| `/sdlc-team:cve <CVE id / finding>` | NIST SSDF vulnerability response: confirm present/reachable/exposed → prioritize (CVSS, EPSS, CISA KEV, SSVC) → remediate → verify → root cause and prevention |
| `/sdlc-team:status [run-id]` | Where a run stands: step, items, open questions, blockers |

**Skill:** `scrum-team`. Claude also loads it when you ask for "the team" or "a squad" to handle something.

**The team** (subagents in `agents/`, each with its own specialist instructions):

| Role | Owns |
|---|---|
| `product-owner` | Value, Sprint Goal, backlog order, scope, final acceptance; comms lead in incidents; risk acceptance for CVEs |
| `business-analyst` | User stories, Given/When/Then acceptance criteria, edge cases; planning lead (timeline) in incidents |
| `architect` | Design, ADRs, subtask breakdown, code review (Google's code review standard) |
| `developer` | Sizing, implementation with unit tests, fixes |
| `security-engineer` | Threat model, security AC, dependency/secret/SAST scans, CVE lead |
| `qa-engineer` | Test plan, black-box acceptance, exploratory and regression testing |
| `qe-engineer` | White-box unit/integration tests, automation, coverage, performance test code |
| `qc-engineer` | Definition of Done; the independent Done / Not done gate |
| `sre-engineer` | NFRs and SLOs, load and system tests, observability, rollout/rollback; ops lead in incidents |

The team is right-sized: a one-line fix uses developer, QC and PO; a story touching auth adds security; anything that runs in production adds SRE.

**Where things go:** every run writes its records to `.sdlc-team/<run-id>/` in your project (brief, backlog with AC and DoD, decisions, test plan and reports, QC report, postmortem or vulnerability record, final report). Add `.sdlc-team/` to `.gitignore` if you don't want it in git.

**How roles talk:** subagents can't call each other, so they post questions, answers, decisions and issues as files on a board in the run folder, and the facilitator relays between them. If you've turned on Claude Code's experimental agent teams, roles can also message each other directly.

**Memory:** the plugin doesn't depend on any memory tool. If your CLAUDE.md or rules set up a long-term memory, the team follows those instructions too.

**Safety:** the team stops for your approval after planning, before anything leaves your working tree (push, PR, deploy), and before any production or shared-environment action. It never fakes test results, and the report lists what didn't run.

**Cost:** one run spawns many subagent turns. A small story is roughly 15–30 turns. Use it for real work items, not one-line questions.

Examples:

```
/sdlc-team:deliver PROJ-142: As a customer I want to reset my password by email so that I can get back into my account
/sdlc-team:deliver Build an internal FAQ chatbot for our HR docs
/sdlc-team:incident checkout API 5xx rate at 12% since 14:05 UTC after the 14:00 deploy
/sdlc-team:cve CVE-2024-45337 in golang.org/x/crypto
/sdlc-team:status
```

Sources: the Scrum Guide 2020, Google's engineering practices (code review), the Google SRE Book and Workbook, NIST SP 800-218 (SSDF), CISA SSVC and KEV, FIRST EPSS and CVSS. The full map is in [`references/sources.md`](plugins/sdlc-team/skills/scrum-team/references/sources.md).

## Repository layout

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json            # catalog of all plugins in this repo
└── plugins/
    └── <plugin-name>/
        ├── .claude-plugin/
        │   └── plugin.json         # plugin name, version, description
        ├── skills/<skill-name>/
        │   ├── SKILL.md            # required: frontmatter + instructions
        │   └── references/         # optional: docs loaded on demand
        ├── commands/<command>.md   # optional: slash commands
        ├── agents/                 # optional: subagents
        └── hooks/hooks.json        # optional: hooks
```

## Adding a new plugin

1. Create `plugins/<plugin-name>/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "<plugin-name>",
     "version": "1.0.0",
     "description": "What it does"
   }
   ```
2. Add skills under `plugins/<plugin-name>/skills/<skill-name>/SKILL.md`, and any commands under `commands/`.
3. Add an entry to `.claude-plugin/marketplace.json`:
   ```json
   {
     "name": "<plugin-name>",
     "source": "./plugins/<plugin-name>",
     "description": "What it does",
     "version": "1.0.0"
   }
   ```
4. Test locally before pushing:
   ```
   /plugin marketplace add ~/workspace/project-learning/claude-plugins
   /plugin install <plugin-name>@sre-devsecops
   ```
5. Bump `version` in both files whenever you change a plugin, so users see the update.

## Rules for third-party content

A skill is instructions Claude will follow, so treat it like code you run:

1. **Trusted source only:** an established repository (thousands of stars, active, clear license).
2. **Read every file before adding it:** `SKILL.md` and everything under `references/` and `scripts/`.
3. **Check for hidden content:** invisible Unicode, HTML comments, unexpected URLs, or instructions to send data anywhere.
   ```bash
   grep -rnP '[\x{200B}-\x{200F}\x{202A}-\x{202E}\x{2060}-\x{2064}\x{FEFF}]' <dir>
   grep -rn '<!--' <dir>
   grep -rhoE 'https?://[^ )"`]+' <dir> | sort -u
   ```
4. **Import single skills, not whole collections.** Overlapping skills compete and give inconsistent answers.
5. **Record the source and the commit you reviewed** in the table below, and keep the original license.

## Third-party sources

| Plugin / skill | Source | Commit reviewed | License |
|---|---|---|---|
| `sdlc-team` | Written for this repository (no third-party content) | — | MIT ([LICENSE](plugins/sdlc-team/LICENSE)) |
| `sre-engineering` / `sre-engineer` | Originally imported from [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills); rewritten in 2.0.0 against Google's SRE Book and SRE Workbook | `4b94d98` | MIT ([LICENSE](plugins/sre-engineering/LICENSE)) |

The 2.0.0 rewrite fixed every known issue from the original import: the slow-burn threshold, undefined recording rules, the percentile-based latency SLI, inconsistent good-event definitions, the CPU throttling query, and the inverted chaos business-hours check. It also fixed errors found by comparing against the books, such as invented error budget tiers, the 30-day window and unsafe automation examples. The full list is in [CHANGELOG.md](plugins/sre-engineering/CHANGELOG.md).
