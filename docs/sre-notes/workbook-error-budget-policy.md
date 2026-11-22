# The Site Reliability Workbook — Appendix B: Example Error Budget Policy

Source: https://sre.google/workbook/error-budget-policy/
Nature of the chapter: a short **example policy document**, not an essay. It is written for a fictional "Example Game Service" and is meant to be adapted as a template. Everything below comes from that one page. Where the page doesn't say something, these notes say so.

## Document metadata (header block, as written)

| Field | Value |
|---|---|
| Status | Published |
| Author | Steven Thurgood |
| Date | 2018-02-19 |
| Reviewers | David Ferguson |
| Approvers | Betsy Beyer |
| Approval Date | 2018-02-20 |
| Revisit Date | 2019-02-01 |

Takeaway: the policy is a governed document. It has a named author, reviewer and approver, and it has a **revisit date** about one year out.

## Core principles

- **Purpose ("Goals")**: there are exactly two. First, "Protect customers from repeated SLO misses". Second, "Provide an incentive to balance reliability with other features".
- **Not punitive ("Non-Goals")**: the policy "is not intended to serve as a punishment for missing SLOs." The freeze is framed as *permission* for the team to focus only on reliability when the data shows reliability matters more than features.
- **Halting change is itself undesirable**. A freeze is a tool to use, not a goal.
- **Error budgets as a control mechanism ("Background")**: they balance reliability against the pace of innovation. Changes cause "roughly 70% of our outages", and feature work competes with stability work.
- **Scope**: in the example, the policy covers **both backend and client releases**. Backend releases ship daily and clients ship weekly.

## Concrete numbers, thresholds and formulas (exact)

- **Error budget = 1 − SLO**. "A 99.9% SLO service has a 0.1% error budget."
- **Worked example**: 1,000,000 requests in four weeks with a 99.9% availability SLO gives a budget of **1,000 errors** over that period.
- **Freeze trigger window**: the error budget is exceeded "for the preceding **four-week** window". This is a rolling 4-week window. It is not a calendar month and not 30 days.
- **Freeze lifts when**: "the service is back within its SLO". There is no fixed freeze duration.
- **Freeze exceptions**: **P0 issues or security fixes** only. Footnote 1 defines P0 as "the highest priority of bug: all hands on deck; drop everything else until this is fixed."
- **Freeze scope**: "all changes and releases". The normal-operation clause says releases "(including data changes)", so data and config changes count as changes.
- **Single-incident threshold**: more than **20%** of the error budget **over four weeks** → a postmortem is mandatory, with **at least one P0 action item** addressing the root cause.
- **Outage-class threshold**: a single *class* of outage consuming more than **20%** of the error budget **over a quarter** → a **P0 item on the quarterly planning document** to fix it **in the following quarter**. Footnote 2: at Google, quarterly planning is public and teams are held accountable to their plans.
- **"Roughly 70%"** of outages are caused by changes (Background).

Note on the two 20% rules: they differ in **unit** (a single incident vs. a class of outage), in **window** (four weeks vs. a quarter) and in **consequence** (a postmortem with a P0 action item vs. a P0 item in quarterly planning).

## Recommended practices and procedures

**SLO Miss Policy**
1. If the service is at or above its SLO, releases proceed under the normal release policy.
2. If the 4-week error budget is exceeded, halt all changes and releases except P0s and security fixes until the service is back within its SLO.
3. Depending on the cause, the team *may* move additional resources from feature work to reliability.

**Team MUST work on reliability if:**
- a code bug or procedural error caused *the service itself* to exceed the budget;
- a postmortem shows an opportunity to soften a hard dependency;
- miscategorized errors failed to consume budget, and counting them would have caused an SLO miss. In other words, the measurement was under-counting.

**Team MAY continue non-reliability feature work if:**
- the outage came from a company-wide networking problem;
- the outage came from a service owned by another team, and that team has frozen its own releases to fix its reliability issues;
- the budget was consumed by users outside the SLO's scope (e.g., load tests or penetration testers);
- miscategorized errors consumed budget even though no users were impacted. In other words, the measurement was over-counting.

**Outage Policy**: see the two 20% rules above.

**Escalation Policy**: if the parties disagree about how the error budget is calculated, or about the actions the policy defines, the issue "should be escalated to the **CTO** to make a decision." This is a single, named, senior tie-breaker, agreed in advance.

## Anti-patterns Google warns against

This page has no explicit anti-pattern list. The warnings below are implied by the text:
- Using the policy as **punishment** for missing SLOs (named as a Non-Goal).
- Treating **halting change as desirable** in itself ("Halting change is undesirable").
- Applying the freeze **blindly, whatever the cause**. The MUST/MAY lists exist so that a team is not frozen for budget burned by another team's outage (if that team has frozen), by company-wide networking, by out-of-scope traffic, or by miscategorized errors that had no user impact.
- **Ignoring measurement errors in either direction**. Under-counting forces reliability work. Over-counting does not force a freeze of feature work.
- Leaving **disputes unresolved**, without a predefined escalation path.

## Template / checklist (section structure of the example)

1. Header: Status, Author, Date, Reviewers, Approvers, Approval Date, Revisit Date
2. **Service Overview**: what the service is, how often releases ship, which release types are covered
3. **Goals**
4. **Non-Goals**
5. **SLO Miss Policy**: trigger, window, freeze scope, exceptions, exit condition, the MUST and MAY conditions
6. **Outage Policy**: the per-incident and per-class consumption thresholds and the actions they trigger
7. **Escalation Policy**: who decides disputes
8. **Background**: boilerplate explaining error budgets (1 − SLO, a worked example)
9. Footnotes defining terms (P0, quarterly planning)

Checklist for adapting it:
- Is the budget window defined (here: rolling four weeks)?
- Is the freeze scope defined (all changes, including data changes)?
- Are exceptions defined (P0s, security fixes)?
- Is the exit condition defined (back within SLO)?
- Are there cause-based MUST/MAY rules?
- Are there postmortem thresholds?
- Is an escalation owner named?
- Is there a revisit date?

## Short verbatim quotes

- "This policy is not intended to serve as a punishment for missing SLOs."
- "Halting change is undesirable"
- "we will halt all changes and releases other than P0 issues or security fixes until the service is back within its SLO."
- "If a single incident consumes more than 20% of error budget over four weeks, then the team must conduct a postmortem."
- "An error budget is 1 minus the SLO of the service."
- "Changes are a major source of instability, representing roughly 70% of our outages"
