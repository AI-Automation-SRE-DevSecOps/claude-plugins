# Chapter 3 — Embracing Risk (Google SRE Book)

Source: https://sre.google/sre-book/embracing-risk/
Authors: Marc Alvidrez (main chapter); "Motivation for Error Budgets" section by Mark Roth. Edited by Kavita Guliani / Carmela Quinito.

## Core principles

- **Past a certain point, more reliability is worse** for the service and its users. It slows feature delivery and raises cost a lot. Users can't tell the difference anyway, because less reliable parts like phones and networks dominate what they experience (intro).
- **Reliability cost is nonlinear.** "An incremental improvement in reliability may cost 100x more than the previous increment" (*Managing Risk*). Two kinds of cost: (1) redundant machine/compute resources; (2) **opportunity cost**, meaning engineers working on risk reduction instead of user-visible features.
- **Risk is a continuum.** SRE gives equal weight to engineering reliability and to finding the right risk tolerance. The goal is to "explicitly align the risk taken by a given service with the risk the business is willing to bear."
- **The availability target is both a minimum and a maximum.** Exceed it, "but not by much." Over-delivering wastes chances to add features, pay down tech debt or cut operating cost.
- **The metric to measure is unplanned downtime** (*Measuring Service Risk*). Other effects of failure (trust, revenue, brand, press) are hard to measure.
- **100% is probably never the right target** (*Key Insights*).
- **An error budget aligns incentives** between product development and SRE and makes uptime a shared responsibility (*Motivation for Error Budgets*).

## Concrete numbers, formulas, tables

The two formulas appear as images on the page. Their standard form is below; the text extraction did not capture them, so they were not re-checked character by character.
- **Time-based availability:** availability = uptime / (uptime + downtime).
- **Aggregate availability:** availability = successful requests / total requests, computed "over a rolling window (i.e., proportion of successful requests over a one-day window)."

Numbers as the chapter states them:
- "Each additional nine corresponds to an order of magnitude improvement toward 100% availability." Examples given: 99.9%, 99.99%, 99.999%.
- **99.99% allows up to 52.56 minutes of downtime per year.** Appendix A has the full table.
- **2.5M requests/day at a 99.99% daily target allows up to 250 errors.**
- A user on a **99% reliable smartphone** can't tell the difference between **99.99% and 99.999%**.
- Google usually sets **quarterly** availability targets and tracks them **weekly, or even daily**.
- Google Apps for Work: a possible **external quarterly target of 99.9%**, backed by a **stronger internal target** and a contract with penalties.
- **Cost/benefit worked example** (*Cost*), assuming every request has equal value:
  - Proposed improvement: 99.9% → 99.99%
  - Increase in availability: 0.09%
  - Service revenue: $1M
  - Value of the improvement: $1M × 0.0009 = **$900**
  - If adding the nine costs less than $900, it is worth it. If it costs more, it is not.
- **Typical ISP background error rate: between 0.01% and 1%.** If the service's error rate is below this, its errors fall within the noise of the user's connection.
- AdSense ads can be served "**hundreds of milliseconds slower**" than AdWords ads. AdWords treats not slowing Search as an **invariant**.
- Bigtable throughput clusters cost "**perhaps as little as 10–50%** of the cost of a low-latency cluster."
- **Error budget example:** an SLO of 99.999% of queries per quarter gives an error budget of a **0.001%** failure rate. An incident that fails **0.0002%** of the quarter's expected queries spends **20%** of the budget.

## Recommended practices and procedures

**Working out risk tolerance for consumer services** (*Identifying the Risk Tolerance of Consumer Services*). SREs work with product owners to turn business goals into objectives they can engineer to. Factors to consider:
1. Target level of availability
2. Types of failures (constant low error rate vs. occasional full outage; partial vs. full outages)
3. Cost (revenue gained per added nine vs. the cost of that nine)
4. Other service metrics, such as latency, which give extra room to take risks

**Infrastructure services** (*Identifying the Risk Tolerance of Infrastructure Services*). Split the infrastructure into **explicitly delineated levels of service**, for example Bigtable low-latency clusters vs. throughput clusters. This pushes the cost difference out to clients, so they choose the cheapest level that meets their needs. The same hardware and software can provide different service classes by changing resources, redundancy, geographic provisioning and, "critically," software **configuration**. Frontend infrastructure (reverse proxies, load balancers) is built for extremely high reliability: "If a request never makes it to the application service frontend server, it is lost."

**Use request success rate rather than uptime for globally distributed services.** Fault isolation means the service is nearly always at least partly "up." The same yield metric works for non-serving systems such as batch, pipeline, storage and transactional systems, counted per unit of work or record.

**Error budget procedure** (*Forming Your Error Budget*):
1. Product Management defines an SLO: the expected uptime per quarter.
2. The monitoring system measures actual uptime as a neutral third party.
3. The difference between the two is the remaining "budget" of unreliability for the quarter.
4. New releases can be pushed while measured uptime is above the SLO, that is, while budget remains.

The two teams set the budget **jointly** and **quarterly**. When the budget is used up, releases are temporarily halted and the effort goes into testing and resilience. Gentler alternatives to on/off ("bang/bang") control include slowing releases or rolling them back as the budget runs low. Network or datacenter outages also count against the budget. If a team struggles to launch features, it may choose to **loosen the SLO**.

**Precondition:** the system only works if the SRE team has "the authority to actually stop launches if the SLO is broken."

## Anti-patterns Google warns against

- Maximizing uptime instead of balancing it against innovation and cost.
- Beating the SLO by a wide margin, which wastes effort.
- Making all infrastructure ultra-reliable for every client, which is "usually far too expensive in practice."
- Treating every failure as equal. Showing private contacts to another user justifies taking the whole service down. A profile picture that fails to render does not.
- Setting the dev/SRE risk balance through negotiating skill, politics, fear or hope. "Hope is not a strategy."
- Counting planned maintenance as unplanned downtime when the business accepts it. The old Ads Frontend counted its maintenance windows as planned downtime.

## Templates and checklists

**Questions for setting a target availability:**
- What level of service will the users expect?
- Does this service tie directly to revenue, either ours or our customers'?
- Is it paid or free?
- What level of service do competitors provide?
- Is it aimed at consumers or enterprises?

**Questions on cost:** What would one more nine add in revenue? Does that extra revenue cover the cost of getting there?

**Typical dev/SRE tensions the error budget helps settle:** software fault tolerance, testing, push frequency, canary duration and size.

## Short verbatim quotes

- "We strive to make a service reliable enough, but no more reliable than it needs to be."
- "We view the availability target as both a minimum and a maximum."
- "100% is probably never the right reliability target"
- "The actual uptime is measured by a neutral third party: our monitoring system."
- "In effect, the product development team becomes self-policing."
- "Hope is not a strategy."
