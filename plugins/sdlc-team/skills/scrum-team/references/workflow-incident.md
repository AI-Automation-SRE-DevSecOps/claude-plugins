# Incident workflow (Google IMAG / SRE)

For an outage, a degraded service, a firing alert or an incident ticket. It follows the SRE Book (ch.14, "Managing Incidents") and the SRE Workbook (ch.9, "Incident Response"). **Stop the impact first, find the root cause later.**

The facilitator is the **Incident Commander (IC)**. It holds every role it hasn't delegated, keeps the incident document, and decides.

| ICS role | Played by |
|---|---|
| Incident Commander | facilitator (main conversation) |
| Operations Lead | `sre-engineer`. Only the Ops lead proposes changes to production, and only the user executes or approves them |
| Communications Lead | `product-owner`: status updates for stakeholders and users |
| Planning Lead | `business-analyst`: tracks the timeline, open questions, follow-ups and handoffs |
| Specialists | `developer` (code and recent changes), `security-engineer` (if it could be a security incident), `architect` (dependencies, design limits) |

## Hard safety rules

- You (and every role) **must not** run anything against production or shared environments (restart, rollback, scale, config change, data change, deleting anything) yourselves. The Ops lead proposes the exact command or action with its expected effect and its blast radius. **The user decides** and runs it, or explicitly tells you to run it.
- Read-only investigation (logs, metrics, dashboards, `kubectl get/describe/logs`, recent deploys, git history) is fine if the user's environment allows it. Say what you looked at.
- If it may be a security incident (data exposure, unknown access, malware), bring in `security-engineer` immediately. Preserve evidence: don't wipe or redeploy over compromised hosts before the user decides.

## Steps

### 1. Declare and assess (IC)

- Create the run (`inc-<slug>-<date>`) and `incident.md` from the template in `artifacts.md`: summary, status, impact, roles, timeline (UTC), current hypothesis, mitigation, next update.
- Assess impact: who and what is affected, since when, how badly (error rate, latency, % of users, data at risk). Use Google's declaration criteria: declare if a second team is needed, if the impact is visible to users, or if the problem isn't solved after an hour of focused work.
- Set a severity (SEV1 = major user impact or data loss; SEV2 = partial or degraded; SEV3 = minor, no user impact) and tell the user.

### 2. Mitigate (Ops lead + specialists, in parallel)

- Ops lead (SRE) proposes **generic mitigations** first, ordered by speed and safety:
  - roll back the most recent change (deploy, config or flag)
  - drain or fail over the bad zone/region/instance
  - add capacity
  - disable the failing feature or flag
  - shed load or rate-limit
  - block a bad client or traffic pattern
  - restore data from backup (last resort)
- Developer checks what changed recently (deploys, merges, config and flag changes, dependency updates) and correlates it with when the impact started.
- Comms lead drafts the first status update: what users see, what we're doing, and when the next update is. The user sends it; you don't contact anyone.
- The IC presents the mitigation options to the user, with the recommendation and the risk of each. The user approves and runs the change, and Ops checks that the impact actually dropped (metrics, error rate, synthetic checks).
- Planning lead keeps the timeline in `incident.md` throughout: every observation, hypothesis, action and result, with a time.
- **Exit:** the impact has stopped or is contained, and this is confirmed by data. Status = mitigated.

### 3. Root cause (developer, SRE, architect, security if relevant)

- Now find out why. Use the troubleshooting method from SRE Book ch.12: observe → hypothesize → test (by trying to disprove) → treat. Prefer evidence to intuition, and note the hypotheses you ruled out.
- Separate the **trigger** (what started it, for example a deploy or a traffic spike) from the **root cause** (why the system was able to fail that way, for example no input validation or a missing timeout). There may be several contributing causes.
- **Exit:** a root cause, or the best supported hypothesis, is written in `incident.md` with its evidence.

### 4. Fix (delivery mini-loop)

- The permanent fix is a backlog item. Run it through `workflow-delivery.md` steps 2–7 at the scale it needs. The AC must include a test that reproduces the failure, and the DoD must include monitoring that would detect it next time.
- Hotfix urgency doesn't skip QC: it shrinks the scope, not the checks.

### 5. Resolve and postmortem

- Resolve when the fix is in place, or when the mitigation is stable and the fix is tracked. Comms lead drafts the final update.
- Write a **blameless postmortem** in `postmortem.md` (template in `artifacts.md`). Write one for every user-visible incident, data loss, manual intervention or long resolution, and whenever the user asks. Focus on systems and processes, never on who did it.
- Action items are typed (**mitigate / prevent / detect / process**), each with one owner role, a priority, and a link to a backlog item. Prefer items that make the failure impossible or automatically detected over items that rely on people being careful.
- If the incident used more than 20% of an SLO's error budget, at least one P0 action item is required (SRE Workbook, error budget policy).
- Retrospective lessons → `report.md`, as described in `collaboration.md`.
