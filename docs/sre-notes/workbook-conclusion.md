# Conclusion (The Site Reliability Workbook)

Source: https://sre.google/workbook/conclusion/
Authors: Dave Rensin, Betsy Beyer, Niall Richard Murphy, Stephen Thorne, Kent Kawahara. Copyright 2018 Google, Inc., published by O'Reilly; CC BY-NC-ND 4.0. Comes after Ch. 21 "Organizational Change Management in SRE" and before Appendix A "Example SLO Document".

**Nature of the chapter:** This is a short essay looking ahead (about 900 words), not a technical chapter. It has no SLO numbers, formulas, alerting thresholds, or procedures. Where this digest lists "practices", they are the authors' observations and encouragements. They are not operating rules.

Section headings: "Onward…" (opening), "The Future Belongs to the Past", "SRE + <Insert Other Discipline>", "Trickles, Streams, and Floods", "SRE Belongs to All of Us", "On Gratitude".

## Core principles

1. **You can adopt SRE in pieces** ("The Future Belongs to the Past"). Small firms are adopting SRE practices "regardless of whether or not they can staff a full-blown, globally distributed SRE team." The authors use a meal metaphor. The *substance and sequence* of the dishes matter, but you don't have to eat the whole meal at once. So partial adoption is legitimate, but the order you adopt practices in still matters.
2. **Large enterprises can adopt SRE quickly.** The authors say the meme that large enterprises are slow to change is "decidedly not true in this case." They expect innovation in SRE from these firms.
3. **SRE principles may carry over to other disciplines** ("SRE + <Insert Other Discipline>"). Security is the leading candidate. The authors note that the term *DevSecOps* is emerging (as of "early spring 2018"). It reflects the recognition that development, security, and operations are interdependent. They openly say they did not cover SRE and security in depth in either book, because "we're not sure what we want to say about it (yet)."
4. **SRE and DevOps are not at odds** ("SRE Belongs to All of Us"). The first SRE book drew heavily on Google's own experience. That "unintentionally alienated some in the larger DevOps community," who felt it ignored other organizations' contributions. The workbook tries to correct this, mainly in Ch. 1, "How SRE Relates to DevOps".
5. **SRE is a global community, and Google is one member of it.** The authors expect non-Google SREs to "significantly outnumber" Google SREs.
6. **The workbook is a starting point** ("Trickles, Streams, and Floods"). It is an implementation companion to the first book. It is also meant to launch new streams of SRE content: books, articles, podcasts, videos.

## Concrete numbers / thresholds / formulas / tables

**None.** The chapter has no SLO targets, error-budget figures, burn rates, tables, or formulas. The only specific facts are:
- Time of writing: "early spring 2018".
- The authors name five editors and authors.
- The prediction window is "over the next year", for innovation from large enterprises.
- The contact for people building SRE products or services is O'Reilly, at bookquestions@oreilly.com.

Don't cite this chapter for any numeric SRE guidance. Use Ch. 2 "Implementing SLOs", Ch. 5 "Alerting on SLOs", and Appendices A and B for that.

## Recommended practices & procedures (as encouraged by the authors)

- **Adopt SRE incrementally** when you can't staff a full SRE team. Pay attention to the order you adopt practices in.
- **Explore SRE applied to security (DevSecOps).** The authors call it "a really good candidate" for SREs looking for an unexplored space.
- **Ask "How would SRE be applied to <insert other discipline>?"** One example from the chapter: a team of lawyers proposed putting **error budgets into nontechnical legal agreements**. That was out of scope for the book, but the authors cite it as evidence of the trend.
- **Market opportunity.** If you're considering offering SRE-adoption products or services, the authors say "the answer is a resounding yes."
- **Join the wider community.** Share and learn through SREcon, meetups, and other publications.
- **Run postmortems on non-incident work too.** The authors say they will run "a lengthy postmortem" on writing the book itself. This shows postmortems used on a project process, not only on outages.

## Anti-patterns Google warns against

- **Anchoring only on your own experience** when you explain or promote SRE. The authors admit doing this in the first book, and it alienated the DevOps community.
- **Treating SRE and DevOps as competitors.** The workbook spends "considerable space" explaining why they are not at odds.
- **Treating SRE as all-or-nothing.** This is implied by the meal metaphor: you don't need a "full-blown, globally distributed SRE team" to start.
- **Assuming large enterprises can't adopt SRE.** The authors call this "decidedly not true".
- **Over-trusting predictions.** The Edgar Fiedler quote is a light warning that forecasting is dangerous.

## Templates / checklists

None in this chapter. The workbook's templates are in **Appendix A (Example SLO Document)**, **Appendix B (Example Error Budget Policy)**, and **Appendix C (Results of Postmortem Analysis)**, which come right after it.

## Short verbatim quotes

- "[h]e who lives by the crystal ball soon learns to eat ground glass." (Edgar Fiedler, quoted in "Onward…")
- "people don't have to eat the whole SRE meal at once." ("The Future Belongs to the Past")
- "development, security, and operations are all interdependent" (definition context for DevSecOps)
- "SRE is now a global community of which Google is one member. It belongs to all of us"
- "This volume is a beginning, not an end."
- "May your queries flow and your pagers stay silent." (closing sign-off)
