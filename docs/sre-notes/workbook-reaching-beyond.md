# SRE Workbook, Chapter 19 — SRE: Reaching Beyond Your Walls

Source: https://sre.google/workbook/reaching-beyond/
Authors: Dave Rensin with Betsy Beyer, Niall Richard Murphy, and Liz Fong-Jones. Part III (Processes). Written about 14 years after Google started practicing SRE.

The chapter argues that a platform team has to practise SRE *with* its customers, internal ones included, and gives a 5-step method for doing it.

## Core principles ("Truths We Hold to Be Self-Evident")

- **Reliability Is the Most Important Feature.** The argument runs: an unreliable system loses its users' trust. Users who don't trust it won't use it when they have a choice. Because of network effects, a system with no users is worth nothing. Closing line: "You are what you measure, so choose your metrics carefully."
- **Your Users, Not Your Monitoring, Decide Your Reliability.** Reliability is what the user experiences. Saying "our monitoring looks fine" doesn't help. The chapter cites the *peak-end rule*: users remember the instability they felt. Monitoring, logs, and alerting are "valuable only insofar as they help you notice problems before your customers do."
- **If You Run a Platform, Then Reliability Is a Partnership.** A UI used only by humans means user-experienced reliability depends almost entirely on your own SRE work. Once you add an API and some users are machines, you are a platform and reliability becomes a partnership. Customers "will hold you accountable for whatever they experience—even if it's not 'your fault.'"
- **Everything Important Eventually Becomes a Platform.** Popular systems end up integrated through APIs. If you refuse to offer one, others will *wrap* your UI into a machine API, "the only difference is that you'll have no control over the outcome."
- **When Your Customers Have a Hard Time, You Have to Slow Down.** Support friction, including StackOverflow and social media, eats into the "innovation budget" and turns into toil. This "*doubly* applies" to internal platform teams.
- **You Will Need to Practice SRE with Your Customers.** One-to-many content (books, blogs, diagrams, videos) is not enough on its own, and it goes stale. You don't have to take customers' pagers. You do have to do "most of the work that normally leads up to pager handoff" with at least a *representative sample* of users.

## Concrete numbers / formulas

- **Composed availability:** customer system at 99% on a platform at 99.999% gives a **best-case** experience of **98.99901%** (0.99 × 0.99999). The product is a ceiling, not an expected value.
- **Monitoring audit:** "up to half of the things your customer is measuring (and alerting on) have zero impact on their SLOs."
- **Measurement period:** after fixing the measurements, "collect data for a month or two."
- **Reality vs. belief:** customers who believe they run at "five 9s" (99.999%) usually measure at **99.5%–99.9%** against their new SLOs.
- **"Earn back the 9s"** example: moving from 99.5% to 99.9%.
- **Selection thresholds** are written as placeholders ("XX%") and left for the reader to set. The chapter gives no specific percentages.

## Recommended procedure — "How to: SRE with Your Customers"

**Step 1: SLOs and SLIs Are How You Speak.** Get customers to measure SLIs, *alert on SLOs*, and share the measurements with you. Without an SLO, a customer reacts to any unexpected change, such as latency moving from T to U, and the conversation goes in circles. With SLOs, they contact you only when the SLO is threatened, for example "burning through our SLO … too quickly", and the conversation uses shared metrics. Exercise: sit down with the customer, explain SLOs, SLIs, and error budgets and how *your* teams practise them, then describe their critical applications in those terms.

**Step 2: Audit the Monitoring and Build Shared Dashboards.** Check whether they are measuring the right things:
1. Find the measurements and alerts that have no SLO impact and get them turned off. That means fewer pages for both sides.
2. Treat the remaining measurements as candidate SLIs and assemble them into SLO calculations.
3. Find the parts of each SLO that no measurement covers, and cover them.
4. Build shared SLO dashboards: you see their application SLOs, and they see your relevant platform data. The goal is that when they report a threatened SLO, little extra information needs to be exchanged.

**Step 3: Measure and Renegotiate.** Collect data for 1–2 months and expect a "rude awakening." Point out that their users "aren't yelling all the time", so they probably never needed five 9s. The key question is how satisfied their users are. If users are happy and nothing suggests better performance would raise adoption, retention, or usage, **you're done**. Ask the question again periodically.

**Step 4: Design Reviews and Risk Analysis.** Review how the customer's application is designed and run, the same way you review internal applications. Look for hidden SPOFs and manual rollouts or rollbacks. **Rank issues by how much error budget each consumes.** Watch which items the customer chooses to fix to "earn back the 9s." This step only happens if the customer still wants improvement after Step 3. The reviews show you:
- how customers consume your platform,
- what reliability mistakes they make,
- which tradeoffs they choose when improving.

They also set realistic expectations, which shape how customers perceive your reliability.

**Step 5: Practice, Practice, Practice.** Run simulated problems together: Wheel of Misfortune, disaster and recovery testing, paper game days. Build shared crisis-communication muscle memory. This builds trust, lowers MTTR, and surfaces edge cases you can turn into platform features. After incidents, run **joint postmortems** rather than just sharing yours.

## Customer selection ("Be Thoughtful and Disciplined")

This model only works for a small percentage of customers. Choose one principled method:

| Approach | Rule | When it fits |
|---|---|---|
| Revenue coverage | Minimum customers accounting for XX% of revenue | Revenue concentrated in a few large customers |
| Feature coverage | Minimum customers covering >XX% of platform features | Diverse platform with a long tail of customers |
| Workload coverage | Sample 1–2 customers per cohort of use case or customer type | Usage dominated by a few distinct use cases, with no dominant single customer |

"Whatever approach you choose, stick to it." Mixing approaches confuses stakeholders and overwhelms the team.

## Anti-patterns

- Telling a customer "our monitoring looks fine; the problem must be on your end."
- Letting break/fix customer support slowly absorb the team's time until the innovation budget is gone.
- Thinking internal platform teams are exempt. They are "likeliest to run into it first."
- Never publishing an API, which leads to uncontrolled UI-wrapping.
- Leaving customers without stated SLOs. They "will inevitably invent one and not tell you until you don't meet it!"
- Customer alerting on metrics that have nothing to do with their SLOs.
- Chasing five 9s that users don't need.
- Only *sharing* postmortems instead of running them jointly.
- Trying to extend deep engagement to every customer, or mixing selection methods.

## Templates / checklists

- **Productive escalation script:** "We're burning through our SLO for application FOO too quickly… SLIs X and Y seem to have fallen off a cliff. They both depend on your API X."
- **Design-review checklist:** hidden SPOFs? manual rollouts or rollbacks? issues ranked by error-budget consumption?
- **Practice menu:** Wheel of Misfortune, DiRT-style testing, paper game days, joint postmortems.

## Short verbatim quotes

- "reliability is the most important feature of any system"
- "Your monitoring, logs, and alerting are valuable only insofar as they help you notice problems before your customers do."
- "in the absence of a stated SLO, your customer will inevitably invent one and not tell you until you don't meet it!"
- "everybody thinks they're getting five 9s"
- "Please don't try extending this model to everyone."
