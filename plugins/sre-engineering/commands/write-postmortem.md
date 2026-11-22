---
description: Draft or review a blameless postmortem with typed, owned action items
argument-hint: <incident description, incident doc, timeline file, or existing postmortem to review>
---

Write or review a postmortem for: $ARGUMENTS

Use the `sre-engineer` skill. Load `references/postmortems.md`, and `references/error-budget-policy.md` to calculate budget impact.

If key facts are missing, ask for them before drafting: impact numbers, the timeline in UTC, the detection method, the mitigation, and the SLO with its window. Mark anything you can't confirm as `TODO(verify)`; don't invent it.

Produce:

1. A postmortem in the template from `references/postmortems.md`, written in blameless language: roles rather than names next to mistakes, and no dramatic wording.
2. A clear split between **trigger** and **root causes**, with contributing factors listed.
3. **Impact**, including the share of the error budget consumed. If it is more than 20% of the 4-week budget, require at least one P0 action item.
4. **Detection analysis**: what fired, how long detection took, and whether an SLO burn-rate alert would have caught it sooner.
5. **Action items**: each with a type (mitigate / prevent / process / other), a priority, one owner placeholder, a bug placeholder and a verifiable end state. Include at least one prevent item. Reject vague items ("improve", "be careful", "train").
6. **Lessons learned**: went well / went wrong / where we got lucky.
7. If you are reviewing an existing postmortem, also give the review checklist results with specific suggested edits.
