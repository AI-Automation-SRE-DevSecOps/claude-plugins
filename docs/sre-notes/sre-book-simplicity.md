# SRE Book, Chapter 9: Simplicity

Source: https://sre.google/sre-book/simplicity/ (page title: "Operational Simplicity: Stability and Agility"). Written by Max Luebbe, edited by Tim Harvey. Epigraph: C.A.R. Hoare, "The price of reliability is the pursuit of the utmost simplicity."

Sections: (intro) · System Stability Versus Agility · The Virtue of Boring · I Won't Give Up My Code! · The "Negative Lines of Code" Metric · Minimal APIs · Modularity · Release Simplicity · A Simple Conclusion

## Core principles

- **Software is dynamic and unstable by nature** (intro). A system is perfectly stable only "in a vacuum": no code changes, no hardware or library changes, no user growth. The SRE job, as summarized in the chapter, is to "keep agility and stability in balance in the system" (a phrase the author credits to his former manager, Johan Anderson).
- **Reliability and agility are not opposed** (System Stability Versus Agility). In SRE's experience, reliable processes tend to *increase* developer agility. Rapid, reliable rollouts make it easier to see production changes, so bugs take less time to find and fix.
- **Boring is a virtue** (The Virtue of Boring). Programs should "stick to the script" and be predictable. "Surprises in production are the nemeses of SRE."
- **Essential vs. accidental complexity** (The Virtue of Boring, citing Fred Brooks, "No Silver Bullet"). *Essential* complexity is part of the problem definition and can't be removed. *Accidental* complexity is "more fluid and can be resolved with engineering effort." The chapter's example: serving web pages quickly is essential complexity, while tuning around Java garbage-collection overhead in a web server is accidental complexity.
- **Every line of code is a liability** for a 24/7 service (I Won't Give Up My Code!). Code should exist only if it has an essential purpose.
- **Simplicity is a prerequisite to reliability** (A Simple Conclusion). Saying "no" to a feature keeps the environment free of distractions, which helps innovation rather than restricting it.

## Concrete numbers / thresholds / formulas / tables

The chapter has **no SLO numbers, thresholds, formulas, or tables**. Its only quantities are illustrations:
- "100 unrelated changes" released at once, the chapter's example of a release whose performance regression is hard to attribute (Release Simplicity).
- "hundreds of lines of commented code" as a source of distraction, and the author's "deleting thousands of lines of code" as satisfying work.
- "24/7" availability expectation as the setting in which every new line of code is a liability.
"Negative lines of code" is a section title and an idea. The chapter gives no formula or target for it.

## Recommended practices & procedures

- **SRE teams should** (The Virtue of Boring):
  1. "Push back when accidental complexity is introduced into the systems for which they are responsible"
  2. "Constantly strive to eliminate complexity in systems they onboard and for which they assume operational responsibility"
- **Delete dead code; rely on source control to bring it back** (I Won't Give Up My Code!). Scrutinize code to make sure it drives business goals, remove dead code routinely, and build "bloat detection into all levels of testing."
- **Be wary of adding features** (Negative Lines of Code). Every line changed or added can introduce defects. Smaller projects are easier to understand and test, and often have fewer defects.
- **Keep APIs minimal** (Minimal APIs). Offer fewer methods and arguments so the API is easier to understand and each method can be made as good as possible. A small, simple API is usually a sign of a well-understood problem.
- **Keep components loosely coupled** (Modularity), both binary-to-binary and binary-to-configuration, so one component's bug can be fixed and pushed independently.
- **Version your APIs** so consumers can stay on the version they depend on and upgrade "in a safe and considered way." This lets each part of the system release on its own cadence instead of requiring a full production push.
- **Give each component a clear, well-scoped purpose.** A distributed system should consist of "collaborators, each of which has a clear and well-scoped purpose."
- **Make data formats modular too.** Use backward- and forward-compatible wire formats; protocol buffers were designed with this as a central goal.
- **Release in small batches** (Release Simplicity). Each change can then be understood in isolation, which lets you "move faster with more confidence." The chapter compares this to gradient descent: small steps, checking whether each one improves or degrades the result.
- **Exploratory coding is allowed with an explicit shelf life** (System Stability Versus Agility). Throwaway code with an expiration date can be looser about tests and release management *because it never ships to production or reaches users*.

## Anti-patterns Google warns against

- **Commenting out code** "so we can easily add it again later". Hundreds of commented lines cause distraction and confusion as files evolve.
- **Gating unused code behind an always-disabled flag** instead of deleting it. This is "a metaphorical time bomb waiting to explode," and the chapter cites the Knight Capital incident (SEC order [Sec13]).
- **"What if we need that code later?"** All three objections in that section are called "terrible suggestions."
- **Emotional attachment to code** that blocks large-scale purges of the source tree.
- **"util" or "misc" binaries.** These are the distributed-system version of a "grab bag" class of unrelated functions.
- **Tight coupling and unversioned API changes.** A single API change can force a rebuild of the entire system and introduce new bugs.
- **Big-batch releases.** Attributing a regression across many simultaneous changes "will take considerable effort or additional instrumentation."
- **Software bloat.** Software grows slower and bigger over time from a constant stream of features.

## Templates / checklists

The chapter has no formal template. Here is an implied review checklist based on its content:
- [ ] Is this complexity essential to the problem, or accidental (fixable with engineering)?
- [ ] Does this code / feature drive a business goal? If not, decline or delete it.
- [ ] Is dead, commented-out, or permanently flag-disabled code being deleted (source control keeps the history)?
- [ ] Do tests at all levels detect bloat?
- [ ] Is the API minimal (few methods and arguments) and versioned?
- [ ] Is the component loosely coupled from other binaries and from its config, with one clear purpose (no "util"/"misc")?
- [ ] Is the data format backward- and forward-compatible?
- [ ] Is the release small enough that each change's impact can be understood on its own?
- [ ] If the code is exploratory, does it have an explicit expiration date and stay out of production?

## Short verbatim quotes

- "The price of reliability is the pursuit of the utmost simplicity." (C.A.R. Hoare, epigraph)
- "At the end of the day, our job is to keep agility and stability in balance in the system."
- "Surprises in production are the nemeses of SRE."
- "Unlike a detective story, the lack of excitement, suspense, and puzzles is actually a desirable property of source code." (Robert Muth)
- "to some extent, every new line of code written is a liability."
- "A small, simple API is usually also a hallmark of a well-understood problem."
- "software simplicity is a prerequisite to reliability."
- "perfection is finally attained not when there is no longer more to add, but when there is no longer anything to take away" (Saint-Exupéry)
