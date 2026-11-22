# The Site Reliability Workbook — Appendix A: Example SLO Document

Source: https://sre.google/workbook/slo-document/ (author Steven Thurgood; CC BY-NC-ND 4.0)

This is a short appendix. It is a worked example of an SLO document for a fictional "Example Game Service", not an essay. Most of the guidance comes from the document's structure and its numbers. The text states very little as explicit rules.

## Document structure (the template)

The sections appear in this order:

1. **Header metadata table**: Status (`Published`), Author, Date (`2018-02-19`), Reviewers, Approvers, Approval Date (`2018-02-20`), Revisit Date (`2019-02-01`).
2. **Service Overview**: the components and how users reach them, plus the measurement window.
3. **SLIs and SLOs**: a table with the columns Category | SLI | SLO, grouped by component.
4. **Rationale**: where the numbers came from.
5. **Error Budget**: how the budget is defined and when the policy is triggered.
6. **Clarifications and Caveats**: measurement blind spots and how errors are classified.

## Core principles

- **Write SLIs as ratios of good events to total events.** Every SLI is "count of X ... divided by count of all ...".
- **Use one measurement window: "The SLO uses a four-week rolling window."**
- **Each component gets SLIs that fit its type.** Request-driven parts (API, HTTP server) get availability and latency. The data pipeline (score pipeline) gets freshness, correctness and completeness.
- **Each objective has its own error budget.** There is no single combined budget.
- **Be honest about the evidence.** The document admits that its numbers are not yet shown to match user experience.
- **SLOs get reviewed again.** The header has an explicit Revisit Date, about one year after approval.

## Concrete numbers, thresholds and formulas (exact)

### Service Overview
- Window: **four-week rolling window**.
- Components: the API (a REST API used by the Android and iPhone apps), the data store, the score pipeline (builds league tables for today, this week and all time), and a public HTTP server.

### SLIs and SLOs table

| Component | Category | SLI (as written) | SLO |
|---|---|---|---|
| API | Availability | count of "api" http_requests which do not have a 5XX status code / count of all "api" http_requests | **97% success** |
| API | Latency | count of "api" http_requests with duration ≤ "0.4" seconds / all | **90% of requests** |
| API | Latency | ... duration ≤ "0.85" seconds / all | **99% of requests** |
| HTTP server | Availability | count of "web" http_requests without 5XX / all "web" http_requests | **99%** |
| HTTP server | Latency | ... duration ≤ "0.2" seconds / all | **90% of requests** |
| HTTP server | Latency | ... duration ≤ "1.0" seconds / all | **99% of requests** |
| Score pipeline | Freshness | data_requests for "api" and "web" with freshness ≤ 1 minute / all data_requests | **90% of reads use data written within the previous 1 minute** |
| Score pipeline | Freshness | ... freshness ≤ 10 minutes / all data_requests | **99% of reads use data written within the previous 10 minutes** |
| Score pipeline | Correctness | count of all data_requests which were correct / count of all data_requests (prober-injected) | **99.99999% of records injected by the prober result in the correct output** |
| Score pipeline | Completeness | pipeline runs that processed 100% of the records / all pipeline runs | **99% of pipeline runs cover 100% of the data** |

Definitions to remember:
- **Availability**: "Any HTTP status other than 500–599 is considered successful." It is measured from the load balancer metrics.
- **Latency** is written as the *proportion of requests that are fast enough*, and each target is a pair of thresholds. It is **not** written as "p90 latency < X ms".
- **Freshness** is measured from the read side, using metrics from the API and HTTP server. It is not measured inside the pipeline.
- **Correctness** uses a prober that "injects synthetic data, with known correct outcomes".
- **Completeness** is defined per hour or per run: "The proportion of hours in which 100% of the games in the data store were processed".

### Rationale (how the numbers were set)
- Availability and latency were based on measurements from **2018-01-01 to 2018-01-28** (four weeks).
- "Availability SLOs were **rounded down to the nearest 1%** and latency SLO timings were **rounded up to the nearest 50 ms**." In both cases the target is made a little looser than what was measured.
- The author picked all other numbers, then checked that the service was already running at or above those levels.

### Error Budget formula
- Budget = **100% minus the goal**, for each objective.
- Worked example: 1,000,000 API requests over four weeks × 3% (100% − 97%) = **30,000 errors**.
- Trigger: the error budget policy (Appendix B) is enacted "when *any* of our objectives has exhausted its error budget".

### Caveats (numbers)
- The correctness prober has "approximately 200 tests, which are injected every 1s". The error budget is **48 errors every four weeks**. Check: 200 × 86,400 × 28 ≈ 483.8M injections, and 483.8M × 10⁻⁷ ≈ 48.

## Recommended practices and procedures

- Give the SLO document a status, an author, reviewers, approvers, an approval date and a revisit date. It should go through review and sign-off like any other governed document.
- Before listing the SLIs, describe the service architecture and the path a user takes through it.
- Name the measurement source for each SLI (load balancer metrics, metrics exported by the pipeline, a prober).
- Write each SLI as a precise formula that anyone can implement.
- Set the first targets from measured historical performance, and round them toward the easier side.
- Use synthetic probers with known correct outcomes to measure correctness, which ordinary traffic cannot measure.
- Connect the SLO document to an error budget policy that states what happens when a budget runs out.
- Write down the measurement blind spots and the rules for classifying errors in a dedicated Caveats section.
- Record the evidence gaps in writing, so later readers can decide whether to spend effort gathering more evidence (footnote 1).

## Anti-patterns (implicit; the appendix rarely warns outright)

- **Measuring only at the load balancer**: "This measurement may fail to accurately measure cases where user requests didn't reach the load balancer." The document names this as a known blind spot.
- **Only 5XX counts as an error**: "We only count HTTP 5XX status messages as error codes; everything else is counted as success." This is a documented simplification. A 4XX caused by the server, for example, would not be counted.
- **Hiding weak evidence**: footnote 1 says it is "necessary to document" this, even when the numbers are not strongly evidence-based.
- **SLOs with no expiry**: the template includes a Revisit Date for this reason.

## Checklist (derived from the template)

- [ ] Metadata: status, author, date, reviewers, approvers, approval date, revisit date
- [ ] Service overview, including the user path and the components
- [ ] Window stated (for example, a four-week rolling window)
- [ ] For each component: category, SLI formula as good/total, measurement source, SLO target
- [ ] Multi-threshold latency and freshness targets where needed (for example, 90% / 99%)
- [ ] Rationale: the measurement period, the rounding rules, and which numbers were picked by judgment
- [ ] Statement of whether the SLOs have been checked against user experience
- [ ] Error budget = 100% − SLO for each objective, with a worked example
- [ ] Link to the error budget policy and its trigger condition
- [ ] Clarifications and caveats: blind spots, error classification, prober details

## Short verbatim quotes

- "The SLO uses a four-week rolling window."
- "Any HTTP status other than 500–599 is considered successful."
- "Availability SLOs were rounded down to the nearest 1% and latency SLO timings were rounded up to the nearest 50 ms."
- "No attempt has yet been made to verify that these numbers correlate strongly with user experience."
- "Each objective has a separate error budget, defined as 100% minus (–) the goal for that objective."
- "We will enact the error budget policy ... when any of our objectives has exhausted its error budget."
