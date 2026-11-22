# The Site Reliability Workbook, Chapter 16: Canarying Releases

Source: https://sre.google/workbook/canarying-releases/. Authors: Alec Warner and Štěpán Davidovič, with Alex Hidalgo, Betsy Beyer, Kyle Smith and Matt Duftler.

## Core principles

- **Definition (intro):** canarying is "a partial and time-limited deployment of a change in a service and its evaluation." The part that gets the change is "the canary". The rest of the service is "the control". The canary is usually a *much* smaller subset of production or users. "Canarying is effectively an A/B testing process."
- **Release engineering principles** (section of the same name): reproducible builds, automated builds, automated tests, automated deployments ("performed by computers, not humans"), and small deployments (small, self-contained changes).
- **A successful release** is defined as "a release made available to customers with no severe defects or SLO violations."
- **Velocity versus reliability** (Balancing Release Velocity and Reliability): 100% reliability is never the right target. In Google's experience, "a majority of incidents are triggered by binary or configuration pushes." Use SLOs and error budgets to measure release impact rather than arguing against change.
- **Decouple components that change at different rates** (Separating Components That Change at Different Rates): binaries, runtime environments (JVM, kernel/OS), libraries, service config/flags, feature/experiment config and user config. Feature-flag frameworks (Gertrude, Feature, PlanOut) separate feature launches from binary releases and let you disable a single bad feature.
- **Why canary** (What Is Canarying?): test environments are never identical to production and tests never cover every scenario, so some defects will reach production. Canarying finds them "with as little impact to your service as possible."
- **Requirements of a canary process:** (1) a way to deploy to a subset of the population, (2) an evaluation process that judges the change "good" or "bad", and (3) integration of that evaluation into the release process. The process proves its value by catching bad candidates with high confidence and passing good ones without false positives.

## Concrete numbers, thresholds and formulas (as written)

- **Example defects** (Our Example Setup): errors on 20% of requests, or 5% of requests taking at least two seconds.
- **Error budget arithmetic** (Minimizing Risk to SLOs and the Error Budget): with a 20% error rate and a 5% canary population, "we serve 20% errors for 5% of traffic, resulting in a 1% overall error rate." The impact on the budget "is directly proportional to the amount of traffic exposed to defects." The budget at risk is "limited by time and the size of the canary population."
- **Model assumptions:** uniform load, the whole error budget available for canaries, only unavailability *introduced by releases* counted, a worst-case 100% failure rate, and overall availability allowed to fall below SLO while the canary runs.
- **Footnote 4:** if the canary's fraction of service equals the gap between actual availability and the SLO, "we can canary nonstop with confidence."
- **Footnote 2:** the fraction of total load sent to the canary should be proportional to the size of the canary population.
- **Number of metrics:** pick the top few, "perhaps no more than a dozen."
- **Cadence example:** if you deploy continuously ("for example, 20 times in a day"), canaries must be much shorter. If you release daily, a canary cannot last a week when only one runs at a time.
- **Metric window rule** (Requirements on Monitoring Data): metric intervals must be "either the same as or less than your canary duration." The counter-example uses an errors-per-hour metric to judge a 30-minute canary on 5% of the population.
- **Figure 16-4** plots errors per version on a base-10 log scale.

## Recommended practices and procedures

- **Break down every metric by canary versus control.** Whole-service monitoring hides a 1% blip (Figures 16-3 and 16-4). If the canary's error rate is too far from the control's, pause and roll back or page a human. If the rates are similar, proceed.
- **Split traffic** with App Engine version splitting, load-balancer backend weights, proxy configs or round-robin DNS.
- **Run only one canary at a time** ("We strongly advise"). Simultaneous canaries add mental load and risk signal contamination.
- **Choose population and duration** by size/duration, traffic volume (more homogeneous requests need less traffic), time of day (performance defects show up mainly under heavy load, so off-peak canaries miss them) and the metrics being evaluated (success rate gives a fast signal; queue depth needs more time or a bigger population). Once you have history, set parameters from "typical canary evaluation failure rates rather than hypothetical worst-case scenarios."
- **Use the simplest model** that meets your technical and business goals. Over-engineering the model leads to "incessant model tuning for no real benefit."
- **Metric selection** (Metrics Should Indicate Problems): start from SLIs. Stack-rank metrics by how well they indicate user-perceivable problems. For frontends, HTTP return codes and latency beat CPU. Exclude 4xx codes such as 404 and add black-box probes for specific URLs. Re-evaluate thresholds regularly as the service evolves.
- **Representative and attributable metrics:** isolate the metric to the process. Whole-machine CPU is poor, per-request process CPU is better, and per-request CPU over the time the process was actually scheduled is better still. Watch for shared failure domains and machine outliers. Tie each canary metric to an SLI.
- **Gradual, multi-stage canary:** start small and judge on the clearest signals (crashes, request failures), then widen the population at later stages so noisier metrics can be evaluated with more confidence.
- **Imperfect isolation** (Dependencies and Isolation): a failed canary is not necessarily the canary's fault, and the canary can degrade the control. Also check absolute measures such as SLOs, not just the A/B delta.
- **Noninteractive systems:** the canary duration must "at minimum span the duration of a single work unit." Keep each work unit within one pool (canary or control) across pipeline stages. Measure end-to-end processing time plus application-specific quality.

## Anti-patterns Google warns against

- Rolling out everywhere at once with no rollback option, then patching forward during the outage.
- Before/after ("canary deployment in time-space") evaluation. Time is "one of the biggest sources of change in observed metrics," for example comparing a weekday with a weekend.
- Too many metrics, or thresholds that are too strict (false positives) or too loose (missed bad canaries).
- Noisy resource metrics such as CPU that cause a "flaky or noisy canary process," which operators then disable or ignore.
- Judging on metric windows longer than the canary.
- Overlapping, concurrent canaries.
- Treating the related techniques (Related Concepts) as full substitutes:
  - Blue/green deployment is effectively a before/after canary and uses twice the resources, unless traffic is split gradually between the two environments.
  - Artificial load gives good code coverage but poor state coverage, and can be dangerous on mutable systems such as billing.
  - Traffic teeing is complex to set up and misleading for stateful systems, for example a shared cache inflating the hit rate.

## Templates and checklists

The chapter has no formal template. Its de facto checklist:
1. Can you deploy to a subset of the population?
2. Is there an automated good/bad evaluation?
3. Is that evaluation wired into the release pipeline?
4. Are metrics broken down by canary and control?
5. Are there no more than about a dozen SLI-derived metrics?
6. Are metric windows no longer than the canary duration?
7. Is only one canary running at a time?
8. Is the duration matched to release cadence and peak load?
9. Are absolute SLO checks in place?

## Verbatim quotes

- "Canarying is effectively an A/B testing process."
- "In Google's experience, a majority of incidents are triggered by binary or configuration pushes."
- "We strongly advise running only one canary deployment at a time."
- "make sure the intervals of your metrics are either the same as or less than your canary duration."
- "Because time is one of the biggest sources of change in observed metrics, it is difficult to assess degradation of performance with before/after evaluation."
