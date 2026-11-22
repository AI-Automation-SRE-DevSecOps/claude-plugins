# Appendix C: Results of Postmortem Analysis

Source: *The Site Reliability Workbook* (O'Reilly, 2018), Appendix C. https://sre.google/workbook/postmortem-analysis/
Page title: "Google SRE - Incident Management: Postmortem Analysis"

**Scope note:** This appendix is very short. It has one paragraph of prose, two data tables, and one sentence linking them. It has no subsections, procedures, checklists or anti-pattern lists. Everything below that the page doesn't say outright is marked **[Interpretation]** so it stays separate from what the source states.

---

## Core principles (as stated)

- **A standard postmortem template is what makes trend analysis possible.** Google uses "a standard postmortem template" (linked to the SRE Book's *Example Postmortem*). The template lets it "consistently capture the incident root cause and trigger."
- **Trigger and root cause are recorded as separate fields.** The page has a separate table for each: Table C-1 for triggers, Table C-2 for root-cause categories.
- **Trend analysis is used to fix systemic problems, not single incidents.** Google uses it "to help us target improvements that address systemic root-cause types." The page gives two examples: "faulty software interface design" and "immature change deployment planning."
- **The data set:** "a sample of thousands of postmortems over the last seven years." Table C-1's caption gives the range as 2010–2017.

## Concrete numbers (exact, as written)

### Table C-1. Top eight outage triggers, 2010–2017

| Trigger | Share |
|---|---|
| Binary push | 37% |
| Configuration push | 31% |
| User behavior change | 9% |
| Processing pipeline | 6% |
| Service provider change | 5% |
| Performance decay | 5% |
| Capacity management | 5% |
| Hardware | 2% |

Arithmetic checks (mine):
- The eight values add up to exactly **100%**.
- **Binary push + configuration push = 68%** of triggers. More than two-thirds of outages were triggered by a change Google pushed itself.
- **Hardware is only 2%**, the smallest trigger listed.

### Table C-2. Top five root-cause categories for outages

| Root-cause category | Share |
|---|---|
| Software | 41.35% |
| Development process failure | 20.23% |
| Complex system behaviors | 16.90% |
| Deployment planning | 6.74% |
| Network failure | 2.75% |

Arithmetic checks (mine):
- These five add up to **87.97%**, not 100%. The rest (about 12%) falls into categories outside the top five, which the page doesn't list.
- Table C-2 uses two decimal places. Table C-1 uses whole percentages.

### Things that are easy to misstate

- "Binary push 37%" and "Configuration push 31%" are **triggers** (what set the outage off). They are **not** root causes. The top root cause is "Software" at 41.35%.
- "Network failure 2.75%" is a **root cause** category. "Hardware 2%" is a **trigger**. The two come from different tables and different taxonomies.
- Table C-1 lists the top **eight** triggers. Table C-2 lists the top **five** root-cause categories.
- The prose says "the last seven years." The caption says "2010–2017." Quote whichever one you are actually citing.
- The data is a **sample** of postmortems ("thousands"). It is not every Google outage.

## Recommended practices and procedures

The page says these directly:
1. Use one **standard postmortem template** across the organization (the SRE Book's *Example Postmortem*).
2. Record both the **trigger** and the **root cause** for every incident.
3. Run **trend analysis** across the postmortem corpus.
4. Use the results to **target systemic improvements** to root-cause *types*.

**[Interpretation]** What practitioners usually take from the data:
- Since 68% of triggers are binary or config pushes, change management gives the most leverage. That means progressive rollouts, canarying (Workbook ch. 16), config validation and fast rollback. Config pushes come close behind binary pushes, so config changes deserve the same rollout rigor as code.
- Keep trigger and root cause as separate, structured fields with a fixed set of categories. Free text can't be aggregated.
- "Development process failure" (20.23%) and "Deployment planning" (6.74%) are process categories. Some reliability work is about process, not just code.

## Anti-patterns

The appendix lists none explicitly. **[Interpretation]** It implies these, by contrast:
- Ad-hoc postmortem formats that can't be compared or aggregated.
- Treating each postmortem as a one-off and never mining the whole corpus for systemic patterns.
- Merging "trigger" and "root cause" into one field. The two tables only work because they are kept separate.

## Templates and checklists

- The page has no template of its own. It links to the SRE Book's **Example Postmortem** (`/sre-book/example-postmortem/`) as Google's "standard postmortem template."
- For postmortem culture and process, see Workbook ch. 10, *Postmortem Culture: Learning from Failure*.

## Verbatim quotes (≤25 words)

- "At Google, we have a standard postmortem template that allows us to consistently capture the incident root cause and trigger, which enables trend analysis."
- "We use this trend analysis to help us target improvements that address systemic root-cause types"
- "based on a sample of thousands of postmortems over the last seven years."
- "Table C-2 presents the top five contributing root-cause categories."
