# The Site Reliability Workbook, Chapter 1: How SRE Relates to DevOps

Source: https://sre.google/workbook/how-sre-relates/
Authors: Niall Richard Murphy, Liz Fong-Jones, Betsy Beyer, with Todd Underwood, Laura Nolan, Dave Rensin.
Tagline: `class SRE implements interface DevOps`

## Core principles

**DevOps (section "Background on DevOps")** is described as "a loose set of practices, guidelines, and culture designed to break down silos." The CA(L)MS acronym (credited to John Willis, Damon Edwards, Jez Humble) stands for Culture, Automation, Lean, Measurement, Sharing. The chapter names five key ideas:
- **No More Silos.** Separate ops and dev teams, siloed knowledge, and purely local optimization hurt the business.
- **Accidents Are Normal.** Failures come from missing safeguards, not only from individual actions. Punishing people pushes them to hide the truth, so "it is more profitable to focus on speeding recovery than preventing accidents."
- **Change Should Be Gradual.** Make changes small and frequent, test them automatically, and roll them back reliably. This leads to CI/CD.
- **Tooling and Culture Are Interrelated.** Culture matters more than tooling: "A good culture can work around broken tooling, but the opposite rarely holds true."
- **Measurement Is Crucial.** Measurement gives an objective basis that different functions agree on.

**SRE (section "Background on SRE").** Ben Treynor Sloss, a Google VP of engineering, coined the term. SRE is "a job role, a set of practices... and some beliefs that animate those practices." It implements part of the DevOps philosophy. Because SRE inherited its culture from Google, it puts less weight on cultural change than DevOps does. Its principles:
- **Operations Is a Software Problem.** Solve it with software engineering methods.
- **Manage by SLOs.** The goal is not 100% availability. Product and SRE teams choose a target together with the business. When an SLO is violated, the teams go "back to the drawing board, blamelessly."
- **Work to Minimize Toil.** If a machine *can* do an operation, a machine often *should*. "Toil is *not* the job." Doing ops work still provides "the wisdom of production," meaning the pages and tickets that show how the system really behaves.
- **Automate This Year's Job Away.** There is a 50% toil limit, explained below.
- **Move Fast by Reducing the Cost of Failure.** The main benefit of SRE is better product development output. Lower MTTR for common faults raises developer velocity.
- **Share Ownership with Developers.** Rigid dev/prod boundaries are counterproductive. Both teams should see the whole stack, and they should "blur the lines": SREs instrument JavaScript, developers qualify kernels.
- **Use the Same Tooling, Regardless of Function or Job Title.**

## Concrete numbers, thresholds, definitions

- **Toil limit: 50%.** "SRE as practiced in Google has a hard limit of how much time a team member can spend on toil, as opposed to engineering that produces lasting value: 50%." Treat it as a **guarantee** of engineering time, not just a **cap**.
- **Murphy-Beyer effect.** After a team automates everything it can, the work that can't be automated is left over. Unless the team acts, that leftover work comes to dominate. At Google the team then takes on more services, up to the limit that still allows 50% engineering time, or moves on to different work.
- **SLO definition (footnote 9).** "A target for performance of a particular metric (e.g., available 99.9% of the time)."
- **Service (footnote 13).** "Software running to perform some business need, generally with availability constraints."
- **SRE expertise areas.** Availability, latency, performance, efficiency, change management, monitoring, emergency response, capacity planning.
- **Ownership default.** Product development teams own their services by default. SRE is "neither available nor warranted for the bulk of services." Onboarding anything requires a Production Readiness Review (footnote 12).
- **Scale example.** A "20-person startup" with one product gains little from being able to withdraw operational support.

## Compare and Contrast

What the two share: both accept that change is necessary; collaboration and shared ownership; small, continual, automatically tested changes; tooling that matters, with "API orientation for system management" valued above any particular implementation; measurement (SLOs drive SRE decisions); blameless postmortems; better velocity as the outcome.

How they differ: DevOps is a wider philosophy and depends more on context. It says little about how to run operations in detail. SRE has narrow responsibilities oriented to services and end users, and it brings an opinionated framework that includes error budgets. SRE says little about siloization. It supports CI/CD because of the better operational practices, not the business case. SRE "believes in the same things as DevOps but for slightly different reasons."

## Recommended practices (section "Organizational Context and Fostering Successful Adoption")

- **Narrow, Rigid Incentives Narrow Your Success.** Don't tie incentives narrowly to launch or reliability outcomes, because metrics get gamed (Goodhart's law). Engage SRE early, ideally at design time.
- **It's Better to Fix It Yourself; Don't Blame Someone Else.** Actively *encourage* engineers to change code and config, and give teams the authority to be radical within their mission. Support blameless postmortems.
- **Allow support withdrawal** from products that are "irredeemably operationally difficult." The threat of withdrawal motivates developers to fix issues. At Google, outright withdrawal "has become institutional."
- **Consider Reliability Work as a Specialized Role.** At Google, SRE and product development are separate organizations. Product development funds SRE headcount growth when a product succeeds. A different org chart isn't required: "a different community of practice" is enough. Give practitioners peer communities and job ladders.
- **When Can Substitute for Whether.** In larger organizations, prioritizing support (supporting X sooner than Y) can play the same role as declining support. Base decisions on objective operational data. Avoid forcing a team to ship before the product is ready.
- **Strive for Parity of Esteem: Career and Financial.** Rate DevOps/SRE and product developers by roughly the same methods, with the same financial incentives.

## Anti-patterns Google warns against

- Treating operations as a cost center (footnote: the British Airways outage in May 2017).
- Finding and punishing the person who made the mistake.
- Monthly change committees that batch changes for human review. That approach is "more or less the opposite of best practice." Higher-risk changes, or changes that can't be validated automatically, still get human vetting (footnote 6).
- Aiming for 100% availability.
- Separate tools for SREs and developers.
- Narrow numeric incentives, and incentives to pass off blame.
- Hiring "DevOps engineers" and expecting immediate benefit: "People sell DevOps, but you can't buy it" (Andrew Clay Shafer).
- **Rename-and-shame.** This means retitling a group DevOps or SRE without changing its organizational position, then shaming the team when nothing improves.
- Power, esteem, or pay imbalances between dev and ops.

## Templates and checklists

The chapter has no formal templates. It works as an adoption checklist: shared incentives, freedom to change code and config, blameless postmortems, a support-withdrawal mechanism, career ladders and peer communities, and parity of esteem and pay. Further reading: *Site Reliability Engineering*, *Effective DevOps*, *The Phoenix Project*, *The Practice of Cloud System Administration Vol. 2*, *Accelerate*.

## Short verbatim quotes

- "class SRE implements interface DevOps"
- "For SRE in the Google context, toil is not the job—it can't be."
- "it's much more useful to think of it as a guarantee"
- "Nothing slows down feature development like losing user data."
- "the practice of rename-and-shame is a hollow one, unlikely to yield benefit."
