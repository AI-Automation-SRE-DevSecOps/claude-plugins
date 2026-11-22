# Chapter 15 – Postmortem Culture: Learning from Failure

Source: https://sre.google/sre-book/postmortem-culture/ (Site Reliability Engineering, O'Reilly 2017). Written by John Lunney and Sue Lueder; edited by Gary O'Connor. Epigraph: "The cost of failure is education." (Devin Carraway)

## Definition (intro)
A postmortem is "a written record of an incident, its impact, the actions taken to mitigate or resolve it, the root cause(s), and the follow-up actions to prevent the incident from recurring." Without a formal learning process, incidents "may recur ad infinitum".

## Core principles

**Google's Postmortem Philosophy**
- Primary goals of writing one: the incident is documented, all contributing root cause(s) are well understood, and, "especially", effective preventive actions are put in place to reduce the likelihood and/or impact of recurrence.
- The chapter does not prescribe a root-cause-analysis method. Teams pick the technique that suits their service (it points to [Roo04]).
- Writing a postmortem is not punishment. It is a learning opportunity for the whole company.
- Postmortems cost time and effort, so Google is "deliberate in choosing when to write one".
- **Blamelessness** is "a tenet of SRE culture". The postmortem looks for contributing causes "without indicting any individual or team" and assumes everyone had good intentions and acted correctly on the information they had.
- Blameless culture came from **healthcare and avionics**, where mistakes can be fatal.
- The focus moves from assigning blame to the systemic reasons someone had incomplete or incorrect information. You fix systems and processes, not people.
- A blameless postmortem still *should* say where and how services can be improved. Blameless does not mean saying nothing critical.

## Concrete numbers / thresholds
The chapter gives **no numeric thresholds**. The triggers say "beyond a certain threshold" and "above some threshold", and each team sets its own values. The only specific figures are anecdotal:
- 2014 TGIF on "The Art of the Postmortem": a release took down a critical service for **four minutes**. The SRE rolled it back immediately and received **two peer bonuses**.
- TGIF is a **weekly** all-hands. "Postmortem of the month" goes out in a **monthly** newsletter.
- Products named as sharing best practices: YouTube, Google Fiber, Gmail, Google Cloud, AdWords, Google Maps.

## Postmortem triggers (Google's Postmortem Philosophy)
Teams have "some internal flexibility", but common triggers are:
1. User-visible downtime or degradation beyond a certain threshold
2. Data loss of any kind
3. On-call engineer intervention (release rollback, rerouting of traffic, etc.)
4. A resolution time above some threshold
5. A monitoring failure (which usually implies manual incident discovery)

Also:
- Define the criteria **before** an incident, so everyone knows when a postmortem is required.
- **Any stakeholder may request a postmortem**, even when no objective trigger fired.
- "Data loss of any kind" is the only trigger with no threshold attached.

## Recommended practices & procedures

**Best Practice: Avoid Blame and Keep It Constructive**
- Taking blame out lets people escalate issues without fear.
- Don't stigmatize a person or team that produces postmortems often.

**Collaborate and Share Knowledge**
- Google writes postmortems in Google Docs from an in-house template (Appendix D, Example Postmortem). Whatever tool you use, it should have: **real-time collaboration**, an **open commenting/annotation system** (crowdsourcing), and **email notifications**.
- Workflow: share the first draft within the team, then have **a group of senior engineers** review it for completeness, then share it more broadly (the larger engineering team or an internal mailing list). The aim is the "widest possible audience that would benefit".
- Postmortems never include information that could identify an end user.

**Best Practice: No Postmortem Left Unreviewed**
- Hold regular review sessions. In them, close out ongoing discussions and comments, capture ideas, and finalize the document's state.
- Once the people involved are satisfied with the document and its action items, add it to a team or organization **repository of past incidents**. Footnote 81 mentions Etsy's *Morgue* as a tool for this.

**Introducing a Postmortem Culture**
- Senior management takes an active part in review and collaboration. Even so, blameless postmortems are "ideally the product of engineer self-motivation".
- Activities: **Postmortem of the month** (newsletter), a **Google+ postmortem group**, **Postmortem reading clubs** (the postmortems discussed are often months or years old), and **Wheel of Misfortune** (new SREs reenact a past postmortem, and the original incident commander attends).
- When people object to the cost:
  - Ease postmortems into the workflow with a trial period of several complete postmortems. This also helps set the trigger criteria.
  - Reward and celebrate good postmortems, both publicly and through individual and team performance management.
  - Get senior leadership to acknowledge and take part.

**Best Practice: Visibly Reward People for Doing the Right Thing**: the TGIF four-minute rollback story (peer bonuses and public applause), plus internal social networks that drive peer praise.

**Best Practice: Ask for Feedback on Postmortem Effectiveness**: regularly survey teams on how well the process works.

**Conclusion and Ongoing Improvements**
- A "Postmortems at Google" working group coordinates templates, builds postmortems automatically from incident-tool data, and extracts data for **trend analysis**.
- Metadata fields were added to the template to support automated analysis. Planned future work is ML to predict weaknesses, support real-time investigation, and reduce duplicate incidents.

## Anti-patterns warned against
- Finger-pointing and shaming. People stop raising issues for fear of punishment, and incidents get "swept under the rug" ([Boy13]).
- Writing the postmortem as "a formality to be forgotten".
- Stigmatizing people or teams that produce many postmortems.
- Leaving postmortems unreviewed.
- Waiting until after an incident to decide whether a postmortem is needed.
- Venting instead of proposing improvements. The chapter's "Pointing fingers" example: "We need to rewrite the entire complicated backend system!… if I get paged one more time I'll rewrite it myself…" Its blameless version proposes the rewrite as an action item and gives the reasons.
- Including end-user identifying information.

## Templates / checklists
**Draft review criteria** (Collaborate and Share Knowledge):
- Was key incident data collected for posterity?
- Are the impact assessments complete?
- Was the root cause sufficiently deep?
- Is the action plan appropriate and are resulting bug fixes at appropriate priority?
- Did we share the outcome with relevant stakeholders?

**Feedback survey questions** (Ask for Feedback):
- Is the culture supporting your work?
- Does writing a postmortem entail too much toil?
- What best practices does your team recommend for other teams?
- What kinds of tools would you like to see developed?

The full template lives in Appendix D (Example Postmortem). This chapter does not reproduce it.

## Short verbatim quotes
- "Writing a postmortem is not punishment—it is a learning opportunity for the entire company."
- "You can't "fix" people, but you can fix systems and processes to better support people making the right choices…"
- "An unreviewed postmortem might as well never have existed."
- "It is important to define postmortem criteria before an incident occurs so that everyone knows when a postmortem is necessary."
- "…it's now a cultural norm that any significant incident is followed by a comprehensive postmortem."
- "…the universal goal of learning from our darkest hours."
