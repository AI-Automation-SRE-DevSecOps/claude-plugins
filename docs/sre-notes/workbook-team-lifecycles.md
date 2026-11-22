# SRE Team Lifecycles (The Site Reliability Workbook, Ch. 20)

Source: https://sre.google/workbook/team-lifecycles/ — David Ferguson and Prashant Labhane with Shylaja Nukala. A roadmap from "unstaffed but aspirational" SRE to globally distributed SRE teams.

## Core principles (the three, stated verbatim)

1. **"SRE needs SLOs with consequences."** (§ SRE Practices Without SREs) SLO performance should guide business decisions.
2. **"SREs must have time to make tomorrow better than today."** (§ Bootstrapping Your First SRE) Without it, toil grows as usage grows. If toil gets too heavy, talented engineers leave.
3. **"SRE teams have the ability to regulate their workload."** (§ Your First SRE Team) The chapter calls this the most subtle of the three. It usually can't be adopted on day one outside a large SRE org, and it is fully asserted only in the Performing stage.

Team building follows **Tuckman's model: forming, storming, norming, performing** (Tuckman 1965).

## Concrete numbers and thresholds (exact)

- Keep **new hires to "less than a third of the team"** so training doesn't overwhelm existing members (§ Storming, New team mitigations).
- Horizontal team tools should have **"a short-term beneficial impact on at least two other teams."**
- Anti-pattern example: team "paged **100 times a day**" and so ignores the pages.
- NYT case study: the goal was **"> 50% project work (versus support-related work)"**. This is NYT's own target, not a Google rule in this chapter.
- New team split: after the split, **"wait three to six months to split the on-call rotations."**
- Geographic split: **12-hour on-call shifts**. Sites **"six to eight hours apart"** work well and avoid **12 a.m. to 6 a.m.** on-call shifts.
- **Three sites** were tried and caused problems. Two sites is the recommended model.
- Travel (§ Finance: Travel budget): every SRE, product dev manager and tech lead visits the other site **annually (at a minimum)**. SRE managers and tech leads visit **at least twice a year**. All SREs convene **at least once a year**.
- Mission Control: product engineers embed in SRE for **six months**. SRE Exchange: **one week**. SRE EDU: **weeklong** immersive training, then a second incident-management series **"a few months later."**
- Planning horizon for choosing where to place the first SRE: the challenges of **"the next 12 months."**
- Norming: stakeholder reports are produced regularly, **"e.g., quarterly."**
- **SRE:dev ratio**: from **~1:5** (low-level infrastructure) to **~1:50** (consumer apps built from many microservices on standard frameworks). Many services sit around **~1:10**.

## Recommended practices and procedures

**Without SREs (§ SRE Practices Without SREs):** accept that you don't want 100% reliability. Set a reasonable SLO that measures what matters most to users. Agree an error budget policy that covers tactical mitigation and longer-term prioritization. Measure the SLO and commit to the policy, which needs leadership agreement. An implicit 100% SLO leaves a team only able to react.

**First SRE (§ Finding/Placing/Bootstrapping):** interview for operations, software engineering, monitoring systems, production automation and system architecture. There are three placement options: in a product dev team, in an ops team, or in a horizontal consulting role. Choose based on your influence, your immediate challenges, the next 12 months (launches point to the dev team, infrastructure changes point to the ops team), your future org plan (if you will centralize, avoid embedding in dev teams) and the person's skills. You can experiment, but stick with **one stable model long term**. Good first projects: monitoring, high-priority postmortem actions, and automating a specific piece of toil. Distributed SREs need a community.

**First team options, least to most complex:** a new team inside a major project, a horizontal SRE team, or converting an existing team. Seed it with internal transfers where possible.

**Mitigations by formation type (§ Risks and mitigations):**
- *Major project:* engage on a single important service, as early as possible (ideally at design). Focus design input on SLOs and reliability risks. Take no operational responsibility on day one. Agree clear onboarding conditions.
- *Horizontal:* seed with respected experts. Deliver tools. Communicate wins. Act as "enablers, not gatekeepers."
- *Converted:* get senior leadership support. Renegotiate responsibilities to create slack. Address job-loss fears head on. Provide training. Change team metrics (align to SLOs) and individual metrics (align to SRE skills). Add an experienced SRE or developer. Give budget or time for new monitoring tools. Review progress regularly.

**Norming maturity checklist:** SLOs and error budgets are in place and the policy is exercised after significant incidents, with leadership interested. On-call is sustainable and **compensated**, with enough tooling, docs and training. Toil is "documented, bounded, and managed." Postmortem culture is established. The team shows most of the DevOps tenets. It runs Wheel of Misfortune and DiRT. Product dev stays in the on-call rotation. It reports to stakeholders regularly (e.g., quarterly). Then pause, celebrate and write a retrospective.

**Performing (§ Self-regulating workload):** partner on all architecture. SRE chooses if and when to onboard a service. When overloaded, reduce the SLO or transfer operational work. If the service can't be run at SLO within the toil constraints, hand it back. Engagement "is not perpetual": find new challenges or hand back deliberately, otherwise attrition follows. With no partner dev team: when out of SLO, stop feature work in favor of reliability work. If the SLO can't be met within toil constraints, lower it unless management adds capacity.

**More teams:** reasons are service complexity, SRE rollout and geographic split. Read earlier postmortems. Seed with your best and highest-potential SREs. Standardize onboarding. Change on-call slowly. Split by architecture, language or location. To avoid orphaned components, give one team "everything not covered" by the other's charter, or appoint a senior cross-team TL. Prioritize rollout by financial or reputational impact and by a minimal viable set of services.

**Parity between offices:** give more tickets to the office that takes fewer pages. Map dev areas to offices. Give Office 2 more internal-improvement projects. Share the interesting projects fairly. Keep size and seniority similar. Split projects across sites. Fund regular travel.

**Many teams (§ Suggested Practices):** Mission Control, SRE Exchange, SRE EDU, horizontal teams for standard tooling, free mobility between teams (transfer to SWE requires having passed the SWE hiring bar), conferences, LCE teams for services that don't justify full SRE, and Production Excellence reviews (pager load, error budget usage, project completion, bug closure rates).

## Anti-patterns

- Renaming "Operations" to "SRE" without first applying SRE practices. A failed rebrand "may poison" the org against SRE.
- An SRE whose work is indistinguishable from other engineering work, or who does more than their fair share of ops work, or who is the only person making config changes. SLOs not taken seriously.
- Spreading too thin. Perfecting the SLO definition while neglecting the service. Dropping SRE principles to hit milestones. Lacking programming skills.
- A horizontal team seen as a "gating" organization.
- Office 2 becoming a "night shift."
- Prioritizing a service for SRE "simply because it's unreliable", or letting devs ignore reliability until SRE engages.
- Office-hours-only on-call (three shifts), which removes the incentive to automate toil.
- Switching models repeatedly. Instability undermines SRE.

## Short verbatim quotes

- "an implicit 100% SLO means a team can only ever be reactive."
- "if toil becomes too burdensome, talented engineers will flee the team."
- "SRE engagement is not perpetual"
- "you should have fewer SREs than the organization would like"
- "Similar to testing or security, reliability is a core pillar of product development, and is funded as such."
