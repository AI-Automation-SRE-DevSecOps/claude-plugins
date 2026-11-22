# claude-plugins

A Claude Code plugin marketplace for SRE and DevSecOps work. Every plugin here has been reviewed before publishing.

## Install

No need to clone anything. In Claude Code, run:

```
/plugin marketplace add AI-Automation-SRE-DevSecOps/claude-plugins
/plugin install sre-skill@sre-devsecops
```

Then restart Claude Code so the new skills and commands are loaded.

You can also run `/plugin` with no arguments to browse and manage plugins from a menu.

### Update

```
/plugin marketplace update sre-devsecops
```

### Uninstall

```
/plugin uninstall sre-skill@sre-devsecops
```

## Plugins

| Plugin | What it does |
|---|---|
| [`sre-skill`](plugins/sre-skill/) | Google SRE practices: SLOs and error budgets, the four golden signals, and alerting philosophy |

### `sre-skill`

**Skill:** `sre-engineer`. Claude loads it automatically for SRE, SLO, monitoring and alerting questions. You can also call it directly with `/sre-skill:sre-engineer`.

**Commands:**

| Command | What it does |
|---|---|
| `/sre-skill:design-monitoring <service>` | Designs a monitoring and alerting system: SLIs/SLOs, golden signals, collection intervals, burn-rate alerts, an alert review table, runbooks and config |
| `/sre-skill:review-alerts [file or dir]` | Reviews existing alert rules against Google's five alert questions and recommends keep / change / downgrade / delete / automate |
| `/sre-skill:define-slo <service>` | Defines SLIs, SLO targets, error budgets, multi-window burn-rate alerts and an error budget policy |

Examples:

```
/sre-skill:design-monitoring checkout API on EKS behind an ALB
/sre-skill:review-alerts terraform/aws/modules/monitoring
/sre-skill:define-slo payments API
```

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
| `sre-skill` / `sre-engineer` | [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) | `4b94d98` | MIT ([LICENSE](plugins/sre-skill/LICENSE)) |

### Changes from the original `sre-engineer`

- Added `references/alerting-philosophy.md`: Google SRE's five alert-review questions, the four rules for pages, symptoms over causes, and a required alert review table.
- `SKILL.md`: links that file, and adds matching MUST / MUST NOT rules and an alerting trigger in the description.
- Added the `design-monitoring`, `review-alerts` and `define-slo` commands.

### Known issues still in `sre-engineer`

These come from the original examples and are not fixed yet:

1. `SKILL.md`: the CPU saturation query divides `throttled_seconds_total` by `cfs_periods_total` (mismatched units). Use `throttled_periods_total / cfs_periods_total`.
2. `SKILL.md`: the slow-burn alert's comment says 6×, but its threshold is `0.001` (1×). It should be `0.006`.
3. `references/monitoring-alerting.md`: the alerts use `error_rate1h`, `6h` and `1d` recording rules that are never defined.
4. `references/slo-sli-management.md`: the latency SLI compares a single p95 value with 500 ms, instead of measuring the fraction of requests under 500 ms.
5. Good events are defined inconsistently: some queries count `4xx` as success, others only `2xx`.
6. `references/incident-chaos.md`: the business-hours check is inverted, and the example kills 50% of pods in `production`.

The `define-slo` command spells out the correct burn-rate windows and SLI definitions, so it works around items 2–5.
