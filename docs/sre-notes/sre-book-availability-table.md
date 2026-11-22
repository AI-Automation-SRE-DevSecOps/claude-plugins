# Appendix A — Availability Table (Google SRE Book, 2017)

Source: https://sre.google/sre-book/availability-table/
Page heading: "Availability Table". The table inside is captioned "Table 1-1. Availability table" (that is the caption on the page, even though this is Appendix A). The appendix is very short: one intro sentence, one table, one paragraph on aggregate metrics, and a pointer to Chapter 3.

## Core principles

1. **Time-based availability is the default framing.** "Availability is generally calculated based on how long a service was unavailable over some period."
2. **The table assumes no planned downtime.** Every figure is the *total* downtime allowed. Planned maintenance would use up the same budget.
3. **Aggregate (request-based) availability is better for real distributed services.** It is "more useful than focusing on outage lengths" in two cases:
   - services that "may be partially available", e.g. multiple replicas where only some are down;
   - services whose "load varies over the course of a day or week rather than remaining constant".
4. **The formulas are not on this page.** It links to the equations "Time-based availability" and "Aggregate availability" in Chapter 3, *Embracing Risk*.

## Concrete numbers: Table 1-1 (exact, as written)

Allowed unavailability window:

| Availability level | per year | per quarter | per month | per week | per day | per hour |
|---|---|---|---|---|---|---|
| 90% | 36.5 days | 9 days | 3 days | 16.8 hours | 2.4 hours | 6 minutes |
| 95% | 18.25 days | 4.5 days | 1.5 days | 8.4 hours | 1.2 hours | 3 minutes |
| 99% | 3.65 days | 21.6 hours | 7.2 hours | 1.68 hours | 14.4 minutes | 36 seconds |
| 99.5% | 1.83 days | 10.8 hours | 3.6 hours | 50.4 minutes | 7.20 minutes | 18 seconds |
| 99.9% | 8.76 hours | 2.16 hours | 43.2 minutes | 10.1 minutes | 1.44 minutes | 3.6 seconds |
| 99.95% | 4.38 hours | 1.08 hours | 21.6 minutes | 5.04 minutes | 43.2 seconds | 1.8 seconds |
| 99.99% | 52.6 minutes | 12.96 minutes | 4.32 minutes | 60.5 seconds | 8.64 seconds | 0.36 seconds |
| 99.999% | 5.26 minutes | 1.30 minutes | 25.9 seconds | 6.05 seconds | 0.87 seconds | 0.04 seconds |

**Period lengths the table assumes.** I worked these out from the numbers; the page does not state them.
- Year = 365 days (90% gives 36.5 days).
- Quarter = 90 days (90% gives 9 days), not 91.25.
- Month = 30 days (90% gives 3 days; 99.9% gives 43.2 minutes). Calendar months of 31 days allow slightly more downtime, and February allows less.
- Week = 7 days, day = 24 hours, hour = 60 minutes.

**Rounding to watch for.**
- 99.999% per day is shown as "0.87 seconds". The exact value is 0.864 s.
- 99.99% per week is "60.5 seconds", about 1 minute.
- 99.9% per week is "10.1 minutes" (exact: 10.08).
- 99.5% per year is "1.83 days" (exact: 1.825).

**Commonly cited figures, checked against the table.**
- "Three nines" (99.9%) = **8.76 hours/year** and **43.2 minutes/month**.
- "Four nines" (99.99%) = **52.6 minutes/year** and **4.32 minutes/month**.
- "Five nines" (99.999%) = **5.26 minutes/year** and **25.9 seconds/month**.
- 99.95% is exactly half the 99.9% budget: 21.6 minutes/month.
- Each extra nine cuts the allowed downtime by 10x.

## Formulas (cross-reference to Chapter 3, not on this page)

This page names two equations in *Embracing Risk* but does not reproduce them:
- **Time-based availability** = uptime / (uptime + downtime)
- **Aggregate availability** = successful requests / total requests

To read the table, multiply (1 − availability) by the length of the period. For example, (1 − 0.999) × 30 days × 1440 min = 43.2 min.

## Recommended practices & procedures

- Use the table to turn an availability target into a concrete downtime budget for the window you actually measure: per month, per quarter, or per week.
- If planned downtime exists, take it out of the same budget. The table's figures assume there is none.
- For replicated or partially available services, and for services with daily or weekly load cycles, measure availability as the share of operations that failed ("X% of all operations failed"), not as outage minutes.
- Go back to Chapter 3 for the actual calculations.

## Anti-patterns (implied by the page)

The page has no explicit "don't" list. The one warning is implied by its preference for aggregate metrics:
- **Using outage length as the only availability measure** for services that can be partly up, or whose traffic changes over time. A short outage at peak traffic can fail more requests than a longer one at a quiet hour, and a partial outage may not count as "down" at all in a time-based measure.

Mistakes readers make that the table itself helps catch (my inference, not Google's text):
- Treating "per month" as a calendar month. The table uses 30 days.
- Forgetting that the "no planned downtime" assumption means maintenance windows count against the budget.

## Templates / checklists

There is no template or checklist on this page. In practice, the table is the reference used to fill in the downtime-budget part of an SLO or error-budget document.

## Verbatim quotes

- "Availability is generally calculated based on how long a service was unavailable over some period."
- "Assuming no planned downtime, Table 1-1 indicates how much downtime is permitted to reach a given availability level."
- "Using an aggregate unavailability metric (i.e., 'X% of all operations failed') is more useful than focusing on outage lengths…"
- "…services whose load varies over the course of a day or week rather than remaining constant."
