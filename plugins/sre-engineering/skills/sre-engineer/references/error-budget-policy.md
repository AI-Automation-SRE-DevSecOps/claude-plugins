# Error Budgets and Error Budget Policy

Sources: SRE Book ch.3 "Embracing Risk", ch.4, App. B "Best Practices for Production Services", App. F (production meeting). SRE Workbook ch.2 "Implementing SLOs", ch.18 "SRE Engagement Model", App. B "Example Error Budget Policy".

## What an error budget is for

- **Budget = 1 − SLO.** At 99.9%, the budget is 0.1%: 1,000 failed requests per 1,000,000 in the window.
- It resolves the conflict between developers and SRE with data. While budget remains, release; once it is spent, focus on reliability. (Book ch.3)
- **The policy is not a punishment.** It is a pre-agreed decision rule that gives the team permission to concentrate on reliability when the data says so. (Workbook App. B)
- Spare budget is meant to be spent on velocity, experiments and planned risk. Don't hoard it. A service that always far exceeds its SLO should release faster or relax its SLO. (Workbook ch.2, ch.18)
- **SRE must have the authority to stop launches** when the budget is gone. Without that, the policy has no teeth. (Book ch.3)

## Budget arithmetic

```
budget_fraction       = 1 − SLO
allowed_bad_events    = budget_fraction × valid_events_in_window
budget_consumed       = bad_events / allowed_bad_events
budget_remaining      = 1 − budget_consumed            (can go negative)
burn_rate             = observed_error_ratio / budget_fraction
time_to_exhaustion    = window_length / burn_rate      (from a full budget)
budget_spent_by_alert = burn_rate × alert_window / window_length
```

Example (Workbook ch.2): a 99.9% SLO with 3,000,000 requests in 4 weeks allows 3,000 errors. An outage causing 1,500 errors uses 50% of the budget.

Incident costing (Workbook ch.2): with a 97% SLO and a 109,897-error budget, a 4-hour bad release (14,066 errors) used 13%. A 20-hour database restore (~72,000 errors) used 65%. With 2–3 bad releases a year against one server failure in 5 years, bad pushes cost about twice as much budget, so fix the release process first.

## Google's example error budget policy (Workbook App. B, adapted)

Use this as the default skeleton. Replace the names, and keep the structure.

```markdown
# Error Budget Policy: <service>
Authors · Reviewers · Approvers (product lead, dev lead, SRE lead) · Approval date · Review date

## Service overview
<one paragraph; link to the SLO document>

## Goals
- Protect users from repeated SLO misses.
- Give an incentive to balance reliability with other features.

## Non-goals
- This policy is not a punishment for missing SLOs. Halting change is undesirable;
  the policy gives teams permission to focus on reliability when the data shows it matters more than features.

## SLO miss policy
If the service has exceeded its error budget for the preceding four-week window:
- Halt all changes and releases, including data changes, except P0 issues and security fixes,
  until the service is back within its SLO.
- Depending on the cause, the team may need to devote all engineering to reliability:
  - Required when the miss came from code or procedural errors made by the team, from a hard dependency
    that could have been a soft one, or from errors that should have counted but weren't measured.
  - Not required when the cause was a company-wide network problem, a dependency owned by another team
    (if that team has frozen its own releases), traffic outside the service's scope, or miscounted
    errors that hit no users.

## Outage policy
- If a single incident consumes more than 20% of the error budget over four weeks, the team must write
  a postmortem with at least one P0 action item to fix the root cause.
- If a single class of outage consumes more than 20% of the error budget over a quarter, the team must
  put a P0 item in the next quarter's planning to address it.

## Escalation policy
If there is disagreement about the calculation or the actions it requires, escalate to the CTO
(or equivalent named executive) for a decision.

## Background
<short explanation of SLOs and error budgets for readers new to them>
```

Key points that are often gotten wrong:
- The trigger is **having exceeded** the budget over the window, not crossing 50% or 25%. Teams may add earlier, softer signals (see below), but the freeze happens only once the budget is exceeded.
- The freeze has **no fixed duration**. It ends when the service is back within SLO. A sample production-meeting note (Book App. F) shows a one-month freeze as one team's choice.
- **Only P0 bug fixes and security fixes** are exempt. A "critical business requirement" is not an exemption; it goes through the escalation path.
- **Data changes are frozen as well as binaries.** Config pushes cause about 31% of outages. (Workbook App. C)
- A dependency outage is a judgement call, decided ahead of time. The Workbook authors still lean towards freezing, because it "will make your users happier". Write the choice into the policy.

## Optional earlier signals

Google's policy doesn't include graduated tiers. If stakeholders want early warnings, add them as **signals, not freezes**:

| Signal | Suggested action |
|---|---|
| Burn-rate ticket fires (10% in 3 days) | Investigate within business hours; raise in the weekly production meeting |
| More than half the budget used before half the window has passed | Give top priority to reliability bugs from the past four weeks; review release risk |
| Budget exceeded | **Policy freeze** (above) |

Whatever you add, name the owner for each action and get the same stakeholder sign-off as the SLO itself.

## Tracking and reporting

- Track the budget daily or weekly. Report SLO status in the weekly production meeting: OK or BAD, value against target, over the trailing window. (Book App. F, ch.31)
- Keep separate budgets for each SLO. The policy applies when **any** one of them is exhausted. (Workbook App. A)
- Engagement ground rules: a quarterly error budget review, and a hard limit on operational work. (Workbook ch.18)

## Prometheus helpers (4-week window)

```promql
# Budget remaining (1 = full, 0 = exhausted, negative = overspent) for a 99.9% SLO.
# Assumes the recording rules job:slo_errors:rate5m and job:slo_requests:rate5m exist (see SKILL.md).
1 - (
  (sum_over_time(job:slo_errors:rate5m[28d]) / sum_over_time(job:slo_requests:rate5m[28d]))
  / 0.001
)

# Current burn rate over 1h (1.0 = spending exactly the budget over the window)
job:slo_errors_per_request:ratio_rate1h / 0.001
```

## Python helper

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class ErrorBudget:
    slo: float              # e.g. 0.999
    window_days: int = 28   # 4-week rolling window

    @property
    def fraction(self) -> float:
        return 1 - self.slo

    def status(self, good: int, valid: int) -> dict:
        if valid == 0:
            return {"sli": None, "note": "no valid events; use synthetic traffic or aggregate services"}
        bad = valid - good
        allowed = self.fraction * valid
        consumed = bad / allowed if allowed else float("inf")
        return {
            "sli": good / valid,
            "allowed_bad": allowed,
            "bad": bad,
            "budget_consumed": consumed,        # 1.0 = fully spent
            "budget_remaining": 1 - consumed,   # negative = overspent
            "policy_freeze": consumed > 1.0,    # Workbook App. B trigger
        }

    def burn_rate(self, error_ratio: float) -> float:
        return error_ratio / self.fraction

    def hours_to_exhaustion(self, error_ratio: float) -> float:
        br = self.burn_rate(error_ratio)
        return float("inf") if br == 0 else self.window_days * 24 / br

eb = ErrorBudget(0.999)
print(eb.status(good=2_998_200, valid=3_000_000))   # 1,800 bad of 3,000 allowed: 60% consumed
print(eb.hours_to_exhaustion(0.0144))               # 14.4x burn: about 46.7 h on a 28-day window
```

## Deploy decisions

Don't encode "business priority overrides an exhausted budget". The decision is:

1. Is the budget for the preceding window exceeded? If not, normal release process.
2. If it is exceeded, is this change a P0 bug fix or a security fix? Then ship it, carefully.
3. If neither, don't ship it. If someone disagrees, use the escalation path in the policy.
