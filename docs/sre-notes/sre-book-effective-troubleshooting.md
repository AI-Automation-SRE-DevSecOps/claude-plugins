# SRE Book, Chapter 12: Effective Troubleshooting (Chris Jones)

Source: https://sre.google/sre-book/effective-troubleshooting/

## Core principles

- **Troubleshooting can be learned and taught.** It is not an innate talent. Doing it well takes two things: (1) a generic troubleshooting process and (2) solid knowledge of the system. Working from first principles alone is "less efficient and less effective" than knowing how the system is supposed to work. (Intro)
- **Hypothetico-deductive method** (Theory): you start from observations and a theoretical model of the system, form hypotheses about the cause, and test them, over and over until you find a root cause. There are two ways to test: (a) compare the observed state against your theories, or (b) actively "treat" the system by changing it in a controlled way and watching what happens.
- **The process (Figure 12-1)** runs Problem Report → Triage → Examine → Diagnose → Test/Treat → Cure. You "needn't always wait" for root-causing or the postmortem before fixing the proximate cause.
- **Stop the bleeding first** (Triage): in a major outage, ignore the urge to hunt for the root cause. Instead "make the system work as well as it can under the circumstances." This is the pilot's rule to "fly the airplane" first [Gaw09].
- **Match the response to the impact** (Triage): one user with a workaround is not an all-hands emergency, but a global outage is.
- **Systems have inertia** (What touched it last): a working system tends to stay working until an external force acts on it, such as a config change or a change in the type of load. Recent changes are a productive place to start.
- **Root cause is often only probable** (Cure): systems are complex, failures can come from several factors acting together, and systems are path-dependent. Reproducing the failure in production may not be possible.

## Concrete numbers, thresholds and examples (exact)

These are illustrative examples from the chapter. Google does not set standards here.

- Log sampling: "one out of every 1,000 operations" (Logging).
- Example log selectors: "`Set` RPCs with a payload size below 1,024 bytes", "operations that took longer than 10 ms to return" (Logging).
- Shakespeare scenario: the alert `Shakespeare-BlackboxProbe_SearchFailure` fires after no results "for the past five minutes". The probe runs once a minute, and "over the past 10 minutes, about half the probes have succeeded". A manual curl returns HTTP **502 (Bad Gateway)**, and the `X-Request-Trace` header lists the backends.
- Negative-result example: a web server handles "~800 connections out of the needed 8,000" before failing from lock contention.
- Spurious-correlation footnote 64: US CS PhDs vs. per-capita cheese consumption, r² = 0.9416 (2000–2009).
- App Engine case study: latency rose "nearly an order of magnitude", and CPU time and serving processes "nearly quadrupled". There was a traffic spike around 20:45, the change happened on a Saturday, and there was a "~250 ms period" before the first RPC. A security scanner created "thousands of whitelist objects over the course of half an hour".
- Footnote 68: RE2 does not backtrack and guarantees linear runtime. PCRE can take exponential time.

## Recommended practices and procedures

**Problem Report**
- A report should state the **expected** behavior, the **actual** behavior, and how to reproduce it if possible.
- Use a consistent form, stored somewhere searchable (a bug tracker). Teams build custom forms or small web apps that automatically generate and route bugs. Offer self-diagnosis or self-repair tools.
- "Open a bug for every issue," even ones that come in by email or IM. Discourage reporting problems directly to a person.

**Triage**
- Emergency options: divert traffic from a broken cluster, drop traffic wholesale to prevent cascading failure, disable subsystems to shed load. If data corruption may be unrecoverable, freeze the system.
- Preserve evidence such as logs while you mitigate.

**Examine**
- Time-series metrics and graphs. Logging: text logs for real-time debugging, structured binary logs for retrospective analysis. Support multiple verbosity levels that can be changed on the fly without a restart, statistical sampling, and a selection language ("show me operations that match X").
- Request tracing (Dapper).
- Expose current state through endpoints: recent RPC samples, per-RPC error and latency histograms, current config. Borgmon can trace a computation back to its source metrics.
- Instrument a client to probe what a component returns.

**Diagnose**
- *Simplify and reduce*: check the data flowing between components with well-defined interfaces. Inject known test data (black-box style). Build a reproducible test case and use non-prod environments for riskier techniques.
- *Divide and conquer*: walk the stack linearly from one end. In very large systems use **bisection**: split the system in half, test the communication paths across the split, and repeat.
- *Ask "what," "where," and "why"*: find what the system is actually doing. This is similar to Five Whys (footnote 67). In the Spanner example, high latency led to CPU saturation, which led to log-sorting, which led to a backtracking regex. The fix: rewrite the regex, look for similar patterns, consider RE2.
- *What touched it last*: keep extensive production logs of deployments and config changes at every layer (binaries down to node packages). Annotate graphs with deployment start and end times (Figure 12-2).
- *Specific diagnoses*: build service-specific tools, but look for commonalities across teams to avoid duplicated effort.

**Test and Treat** (test design considerations)
1. An ideal test has mutually exclusive alternatives, so it rules some hypotheses in and others out.
2. Consider the obvious first. Test in decreasing order of likelihood, weighed against the risk each test poses to the system.
3. Watch for confounding factors (for example, a firewall that allows pings only from the app server's IP).
4. Active tests have side effects (more CPUs can expose data races, verbose logging can make latency worse).
5. Some tests are only suggestive (race conditions and deadlocks).
- Take clear notes of ideas, tests and results. A shared doc or chat gives you timestamps for the postmortem (footnote 70). Make active changes systematically and document them so you can roll back to the pre-test setup.

**Negative Results Are Magic** (sidebar, Randall Bosetti)
- Negative results are conclusive and valuable. Publish them, especially surprising ones. Tools built for a failed experiment (for example, Apache Bench) outlive it. Be skeptical of documents that never mention failure.

**Cure**: write a postmortem covering what went wrong, how you tracked it down, how you fixed it, and how to prevent a recurrence.

**Making Troubleshooting Easier**
- Build observability into every component from the start: white-box metrics plus structured logs.
- Design well-understood, observable interfaces between components.
- Use a unique request ID across all the RPCs a request generates.
- Simplify, control and log code and environment changes.

## Anti-patterns (Common Pitfalls and elsewhere)

- Looking at irrelevant symptoms or misreading metrics, which leads to wild goose chases.
- Not knowing how to change the system, its inputs or its environment safely to test hypotheses.
- Wildly improbable theories, or latching onto past causes ("it happened once, so it's happening again").
- Chasing spurious correlations, whether coincidences or effects of a shared cause (packet loss and failed disks, both caused by a power outage).
- Root-causing during a major outage instead of mitigating.
- Reporting problems directly to a person instead of filing a bug.
- Leaving the system in an "unknown hodge-podge configuration" after tests.
- Not reporting negative results.
- (Case study) An in-memory config cache whose per-request cost scales with the number of objects (a linear `for` loop, footnote 76).

Heuristics: think "horses not zebras", prefer simpler explanations (Occam's Razor), but remember Hickam's dictum: several common low-grade problems may explain the symptoms (footnote 62).

## Templates and checklists

- **Problem report:** expected behavior / actual behavior / reproduction steps. File it in the tracker and route it to on-call.
- **Diagnosis loop:** Triage (mitigate, preserve evidence) → Examine (metrics, logs, traces, state endpoints) → Diagnose (simplify, bisect, what/where/why, recent changes) → Test (5 considerations above, keep notes) → Cure (postmortem).

## Case study takeaways (App Engine)

- The `merge_join` correlation was spurious. It was disproven when static content turned out to be slow too.
- The team mitigated first: they moved to the most CPU-rich instance type so a launch the next week could go ahead, and investigated afterwards (footnote 75).
- The root cause was a long-standing access-control bug plus a security scanner that created whitelist objects, which were then checked on every request.

## Short verbatim quotes

- "Your first response in a major outage may be to start troubleshooting and try to find a root cause as quickly as possible. Ignore that instinct!"
- "Stopping the bleeding should be your first priority; you aren't helping your users if the system dies while you're root-causing."
- "when you hear hoofbeats, think of horses not zebras."
- "a working computer system tends to remain in motion until acted upon by an external force"
- "Experiments with negative results are conclusive."
- "I don't know where the fire is yet, but I'm blinded by smoke coming from this whitelist cache."
