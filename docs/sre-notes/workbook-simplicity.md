# SRE Workbook, Chapter 7: Simplicity

Source: https://sre.google/workbook/simplicity/
Authors: John Lunney, Robert van Gent, Scott Ritchie, with Diane Bates and Niall Richard Murphy. The epigraph is Gall's Law.

## Core principles

- **Simplicity correlates with reliability.** Simple software breaks less often and is faster to fix. Simple systems are easier to understand, maintain and test (intro).
- **Simplicity is end-to-end.** It covers code, system architecture, and the tools and processes that manage the software lifecycle (intro; "Simplicity Is End-to-End, and SREs Are Good for That").
- **Complexity grows unless someone pushes back.** The chapter says there is "no serious opposition" to two claims: complexity will increase in living systems "unless there is a countervailing effort", and providing that effort is worthwhile ("Measuring Complexity").
- **Complexity is an externality.** Its cost usually falls on the people who work in and around the system later, not on whoever added it. So someone needs to champion end-to-end simplicity, and SREs suit the role because they treat the system as a whole.
- **Simplification is a feature, and simplification is efficiency.** It saves engineering time and cognitive load rather than compute ("Regaining Simplicity").
- **Most simplification means removing things.** Sometimes that is a direct removal, such as an unused remote data dependency. Sometimes it takes a redesign, such as fetching shared remote data once and forwarding it instead of fetching it twice.

## Concrete numbers, definitions, thresholds

- **Cyclomatic complexity (CCN)** measures "the number of distinct code paths through a specific set of statements." A block with no loops or conditionals has **CCN = 1**. Tools exist for Visual Studio, Eclipse and IntelliJ. The hard part is not the measuring. It is telling necessary complexity from accidental complexity, seeing how one method affects the wider system, and choosing how to refactor.
- **System complexity**: formal methods are rare. Counting entities (such as microservices) and the communication paths between them grows "hopelessly large very quickly" for sizable systems.
- **Practical proxies for system-level complexity** ("Measuring Complexity"):

| Proxy | Question |
|---|---|
| Training time | How long before a new team member can go on-call? (Poor docs add subjective complexity.) |
| Explanation time | How long does it take to explain a comprehensive high-level view, such as a whiteboard architecture diagram with each component's dependencies? |
| Administrative diversity | How many ways are there to configure similar settings? Is config centralized or spread across locations? |
| Diversity of deployed configurations | How many unique configurations run in production (binaries, binary versions, flags, environments)? |
| Age | How old is the system? (Hyrum's Law: over time, API users depend on every aspect of the implementation.) |

- **10% time**: as an example, "reserve 10% of engineering project time for 'simplicity' projects." Footnote 5 adds that this "doesn't mean the team has a green light to introduce complexity with the other 90%."
- Case Study 4 opens with "Over the past 15 years". This is background, not a threshold.

## Recommended practices and procedures

- **Reader actions** (the chapter's explicit callouts):
  1. Before an engineer's first on-call shift, have them draw (and redraw) system diagrams. Keep a canonical set of diagrams in the docs.
  2. Make sure an SRE reviews all major design docs, and that the docs show how the design affects the system architecture. The SRE can suggest simpler alternatives.
  3. Have engineers brainstorm the system's known complexities and ideas for reducing them.
- **Leadership**: celebrate and explicitly prioritize simplification. Treat it like a feature launch, and "measure and celebrate code addition and removal equally." Example: Google's "Zombie Code Slayer" badge for engineers who delete significant amounts of code. Staff simplification work, reserve SRE time for it, and make it count for careers.
- **Team structure**: if SRE teams split, consider "a small rotating group of SREs" who keep working knowledge of the whole stack (likely with less depth) and push for conformity across it.
- **What to look for when diagramming**:
  - *Amplification*: retries at several levels multiply the total number of RPCs.
  - *Cyclic dependencies*: a component that depends on itself, often indirectly, can make a cold start impossible.
- **Rewrites (Case Study 2, Borg to Omega)**: think about the full project lifecycle, which includes a moving target, a full migration plan, and the cost of running both systems during migration. Compare the expected result against what the current system *would look like with the same improvement effort*, not against the system as it is today. Wide APIs with many users are very hard to migrate. In the end, Omega's ideas went back into Borg and helped jump-start Kubernetes.
- **API design (Case Study 1)**: prefer structured types (Protocol Buffers, Thrift) over a generic key/value "bag". They look more complex but give simpler end-to-end solutions, because they force design decisions and documentation up front.
- **Standardization (Case Study 3, Display Ads)**: SREs wrote uniformity standards: one way to copy large data sets, one way to do external data lookups, and common templates for monitoring, provisioning and configuration. They added shared logic behind flags, later removed the flags, and consolidated into fewer backends. Integrate acquired or running systems incrementally.
- **Shared platform (Case Study 4)**: converge bespoke production stacks onto one microservices platform, run by one SRE group. Services are "*managed*, not *hosted*", and teams keep responsibility for them. New services must use the platform, and legacy services must migrate or be phased out. The outcome was that developer teams run hundreds of services without deep SRE engagement, and **tiered SRE engagement** became common (from light consulting and design reviews to shared on-call). Aim for incremental wins at each stage.
- **Dependencies (Case Study 5, pDNS)**: pDNS depended on itself through its load balancer and Svelte. The fix was to cache nearby Svelte IPs in local storage on all production machines, and to whitelist which services may talk to pDNS, shrinking that list over time. The lesson: "use an explicit whitelist to prevent accidental additions."

## Anti-patterns Google warns against

- Overly generic APIs, such as the "bag" of key/value pairs. Keys go undocumented and compatibility breaks.
- Wholesale rewrites justified by comparing a clean new design with today's messy system. The grass isn't always greener.
- Retries layered at several levels, which cause amplification. Retries in one component can overload a database.
- Circular or transitive self-dependencies, which block a cold start.
- Redundant lookups within one request. The chapter calls this a "system smell", the system-level version of a code smell.
- Bespoke per-team production stacks (CI/CD, monitoring). They are costly and make it harder to move services or engineers between teams.
- Asking people for one huge refactor that only pays off at the very end.
- Splitting SRE teams so narrowly that nobody can drive cross-stack simplification.
- Treating the 10% simplicity budget as permission to add complexity with the other 90%.

## Checklist (derived from the chapter)

- [ ] Track the proxies: training time, explanation time, administrative diversity, deployed-config diversity, age.
- [ ] Keep canonical architecture diagrams, and have new on-callers draw them.
- [ ] Have an SRE review every major design doc for its architectural impact.
- [ ] Check diagrams for retry amplification and cyclic dependencies.
- [ ] Whitelist the clients of critical services.
- [ ] Reserve time (for example 10%) for simplification, and reward code deletion.
- [ ] For a rewrite, cost the moving target, the migration and the dual-running period.

## Short verbatim quotes

- "A complex system that works is invariably found to have evolved from a simple system that worked." (Gall's Law, epigraph)
- "In economic terms, complexity is an externality."
- "Simplification is a feature."
- "Redundant lookups in a single request represent a 'system smell.'"
- "We were like cave dwellers who could only light fires by running with a torch lit from the last campfire."
- Dijkstra (footnote 4): lines of code should be regarded "not as 'lines produced' but as 'lines spent.'"
