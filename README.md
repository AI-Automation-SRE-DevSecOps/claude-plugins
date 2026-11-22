# claude-plugins

A Claude Code plugin marketplace for SRE and DevSecOps work. Every plugin here has been reviewed before publishing.

## Install

No need to clone anything. In Claude Code, run:

```
/plugin marketplace add AI-Automation-SRE-DevSecOps/claude-plugins
/plugin install sre-engineering@sre-devsecops
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
| `sre-engineering` / `sre-engineer` | Originally imported from [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills); rewritten in 2.0.0 against Google's SRE Book and SRE Workbook | `4b94d98` | MIT ([LICENSE](plugins/sre-engineering/LICENSE)) |

The 2.0.0 rewrite fixed every known issue from the original import: the slow-burn threshold, undefined recording rules, the percentile-based latency SLI, inconsistent good-event definitions, the CPU throttling query, and the inverted chaos business-hours check. It also fixed errors found by comparing against the books, such as invented error budget tiers, the 30-day window and unsafe automation examples. The full list is in [CHANGELOG.md](plugins/sre-engineering/CHANGELOG.md).
