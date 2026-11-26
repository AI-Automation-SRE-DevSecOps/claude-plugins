# Sources

What each part of this skill is based on.

| Topic | Source | Used for |
|---|---|---|
| Scrum | [The Scrum Guide (2020)](https://scrumguides.org/scrum-guide.html) | Accountabilities (PO is one person and owns the backlog; developers size the work), events, Product Goal, Sprint Goal, Definition of Done, Increment ("work cannot be considered part of an Increment unless it meets the Definition of Done"), refinement |
| User stories | Mike Cohn, *User Stories Applied*; Bill Wake, INVEST (2003) | Story format, INVEST check |
| Acceptance criteria | Dan North, *Introducing BDD*; Gherkin (Cucumber) | Given/When/Then |
| Code review | [Google Engineering Practices: The Standard of Code Review](https://google.github.io/eng-practices/review/reviewer/standard.html), [What to look for](https://google.github.io/eng-practices/review/reviewer/looking-for.html) | Approve when code health improves; design first; `Nit:` |
| Testing | Google SRE Book ch.17 *Testing for Reliability*; Google Testing Blog (test sizes); ISTQB terminology (white-box, black-box, exploratory) | Test levels, white-box vs black-box, system and performance tests |
| Incident response | [SRE Book ch.14 *Managing Incidents*](https://sre.google/sre-book/managing-incidents/), [SRE Workbook ch.9 *Incident Response*](https://sre.google/workbook/incident-response/), SRE Book ch.12 *Effective Troubleshooting* | IC/Ops/Comms/Planning roles, 3Cs, declaring, mitigate first, generic mitigations, troubleshooting method |
| Postmortems | [SRE Book ch.15 *Postmortem Culture*](https://sre.google/sre-book/postmortem-culture/), SRE Workbook ch.10 | Blameless postmortems, trigger vs root cause, typed and owned action items |
| Error budgets | SRE Workbook App. B *Example Error Budget Policy* | >20% of budget in one incident → P0 action item |
| Launch / readiness | SRE Book ch.27 and App. E *Launch Coordination Checklist* | Operational readiness, rollout and rollback |
| Secure development | [NIST SP 800-218 SSDF v1.1](https://csrc.nist.gov/pubs/sp/800/218/final) | RV.1 identify and confirm, RV.2 assess, prioritize and remediate, RV.3 root causes |
| Vulnerability priority | [CISA SSVC](https://www.cisa.gov/stakeholder-specific-vulnerability-categorization-ssvc), [FIRST EPSS](https://www.first.org/epss/), [FIRST CVSS](https://www.first.org/cvss/), [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) | Track/Track*/Attend/Act, exploitation probability, severity, known exploitation |
| Not-affected statements | CISA VEX minimum requirements; OpenVEX | VEX justifications |
| Threat modeling | Microsoft STRIDE; OWASP Threat Modeling, OWASP ASVS, OWASP Top 10 | Security review |
| Multi-agent pattern | Anthropic `feature-dev` plugin (code-explorer, code-architect, code-reviewer agents); [Claude Code subagents](https://code.claude.com/docs/en/sub-agents) and [agent teams](https://code.claude.com/docs/en/agent-teams) docs | Facilitator in the main thread, role agents, parallel spawns, continuing agents with SendMessage |

Numbers in this skill come from these sources. Where a team's own policy differs (remediation SLAs, sprint length, coverage thresholds), use the team's policy and say so.
