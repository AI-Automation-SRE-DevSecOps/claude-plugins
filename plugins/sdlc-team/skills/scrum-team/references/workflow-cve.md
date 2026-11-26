# Vulnerability response workflow (CVE)

For a CVE id, a security advisory, a scanner finding or a vulnerable dependency. It follows NIST SP 800-218 (SSDF) practice group **RV, Respond to Vulnerabilities**: RV.1 identify and confirm, RV.2 assess, prioritize and remediate, RV.3 analyze root causes. Prioritization uses CVSS (severity), FIRST EPSS (likelihood of exploitation), the CISA KEV catalog (known exploitation) and CISA SSVC (the decision).

Lead role: **security-engineer**. Others: **developer** (code and dependency changes), **architect** (impact on design, breaking changes), **qe-engineer** and **qa-engineer** (regression), **sre-engineer** (exposure, deployment, runtime mitigations), **product-owner** (risk acceptance and priority against other work), **qc-engineer** (gate).

## Safety rules

- Work only on the user's own code and systems. Never write or run exploit code against systems the user doesn't own or isn't authorized to test. A reproduction, when needed, is a minimal local test case.
- Don't publish details of an unfixed vulnerability (public issues, PR descriptions, commit messages that explain the exploit) without the user's approval. Coordinated disclosure is the user's decision.
- Production changes (WAF rules, config, deploys) follow the same rule as incidents: propose, and the user approves.

## Steps

### 1. Identify and confirm (RV.1)

Security engineer, with the developer:

- **Read the advisory** from primary sources: the NVD entry, the vendor or project advisory, the GitHub Advisory Database or OSV. Note the affected package, the affected and fixed version ranges, the vulnerable function or configuration, the preconditions, and the CWE. If you can't reach these sources, ask the user for the advisory text; don't guess version ranges.
- **Find where it is.** Lockfiles and manifests (`package-lock.json`, `yarn.lock`, `poetry.lock`, `requirements*.txt`, `go.sum`, `pom.xml`, `Gemfile.lock`, `Cargo.lock`), container images and base images, IaC, vendored code, and SBOMs if present. Run whatever scanner the repo has (`npm audit`, `pip-audit`, `govulncheck`, `osv-scanner`, `trivy`, `grype`) and say which ran. Include transitive dependencies and the path that pulls them in.
- **Confirm it's exploitable here**:
  - Present: is an affected version actually installed or shipped?
  - Reachable: is the vulnerable code called, or is the vulnerable feature or config enabled? `govulncheck` gives call-graph reachability; elsewhere, grep for the API.
  - Exposed: can an attacker reach that path (internet-facing, authenticated only, internal only, build-time only)? SRE helps with exposure.
- **Outcome**: *affected*, *not affected* (with a reason), or *under investigation*. Write it in `vuln.md` (template in `artifacts.md`).
- If not affected, write the VEX-style justification:
  - `component_not_present`
  - `vulnerable_code_not_present`
  - `vulnerable_code_not_in_execute_path`
  - `vulnerable_code_cannot_be_controlled_by_adversary`
  - `inline_mitigations_already_exist`

  Then go to step 5 with a "no change" decision that the PO signs off.

### 2. Assess and prioritize (RV.2)

Security engineer proposes and the product owner decides:

- Record: CVSS base score and vector (from the advisory), the **EPSS** probability and percentile (FIRST publishes it daily; the 30-day exploitation probability), and whether the CVE is in the **CISA KEV** catalog. If you can't look these up, say so and ask the user. Don't make up scores.
- Evaluate the SSVC decision points for this deployment:
  - **Exploitation**: none / public PoC / active. Being in KEV counts as active.
  - **Automatable**: can an attacker exploit it at scale without human steps?
  - **Technical impact**: partial / total.
  - **Mission prevalence and public well-being impact** of the affected system.
- Map them to the CISA SSVC outcome **Track / Track\* / Attend / Act**:
  - Act: fix as soon as possible, leadership aware.
  - Attend: sooner than normal timelines.
  - Track / Track\*: normal update timelines (Track\*: watch closely).
- Use the user's organisation's remediation SLA for the outcome if they have one. Ask if unknown. Known exploited (KEV) plus exposed is always the most urgent class.
- Choose the response:
  1. **upgrade** to the lowest fixed version that is compatible
  2. **patch or backport** the fix
  3. **mitigate** with a vendor workaround or config change
  4. **compensating control** (WAF rule, network restriction, disable the feature) while the fix is prepared
  5. **accept** the risk: the PO decides, with a reason and an expiry date

  Record the decision in `decisions.md`.

### 3. Remediate (delivery mini-loop)

- Make the change as a backlog item and run `workflow-delivery.md` steps 2–6.
- The AC always include:
  - the scanner or version check no longer reports the CVE
  - the reachability path is gone or protected
  - existing tests pass
  - no unintended major-version jump
- The DoD always includes:
  - the lockfile is updated
  - the changelog or release notes of the upgraded package were reviewed for breaking changes (architect)
  - a regression test exists where a minimal local test case is possible
- QE and QA run the regression suite, and focus on what the upgraded component does.
- SRE writes the rollout and rollback plan, and checks that runtime mitigations (if any) can be removed after the fix ships.

### 4. Verify

- Re-run the scanner and show the before/after output in `vuln.md`.
- QC gate: every AC and DoD line has evidence.
- For mitigations and compensating controls, verify they actually block the attack path in a test environment, and record the evidence.

### 5. Root cause and prevention (RV.3)

- Why was the vulnerable version there, and why wasn't it caught sooner? For example: no automated dependency updates, scanning not in CI, pinned to an old major version, an unused dependency, a base image not rebuilt.
- Look for the same class elsewhere: other services and images with the same package, or other uses of the same unsafe pattern (the CWE).
- Prevention items go to the backlog, typed like postmortem items:
  - dependency update bot (Dependabot or Renovate)
  - scanner gate in CI with an agreed severity threshold
  - SBOM generation
  - base image rebuild schedule
  - removing unused dependencies
  - secure-coding rule for the CWE
- `report.md` summarises: status (fixed / mitigated / not affected / accepted), the evidence, the decision and who made it, and the follow-ups.
