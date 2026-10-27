# Recipes

Multi-step flows that answer common questions. Each one goes from a count to
the records behind it. Adjust the period and filters to fit the question.

## A team's numbers, combined and per person

1. The team: `find_agents agent_id=[ids]`, or by name from the user. Never an
   empty list.
2. Calls, meetings, leads, speed to lead take the team as a list
   (`agent=[names]`, `owner=[names]`): one call for the total, one with
   `group_by=agent` / `owner` for the rows per person.
3. Deals take one owner: `pipeline_stats(group_by=owner)` once, keep the team's
   rows by `group` (the agent_id), combine as metrics.md says.
4. Mark each per-person value resting on fewer than 5 records as thin, and do
   not rank people on thin values.

## What to coach one agent on

1. `call_scorecard(agent=[name], outcome=["connected"], P)`: the weakest
   quality is the one with the lowest best-label share.
2. `search_calls(agent=[name], filters={dimension: "<weakest>", label:
   "<worst label>"}, sort_by=score, sort_order=asc, page_size=5)`: the examples.
3. `get_call_grading(call_id)` for each: the reasoning, the quote, the next
   actions. That is the evidence to coach from.
4. The same with speed to lead:
   `search_speed_to_lead(owner=[name], verdict=["missed","not_reached"])`.

## Why deals are lost

1. `pipeline_stats(closed P, status=["lost"], group_by=lost_reason)`: the
   reasons by count. `lost_reason_subcategory` goes one level deeper.
2. `pipeline_stats(..., group_by=owner, filters={main_cause: "individual"})`:
   losses down to how the deal was handled, per owner.
3. `search_deals(status=["lost"], closed_period=P, filters={main_cause:
   "individual"}, sort_by=amount, sort_order=desc, page_size=5)`, then
   `get_deal_verdict` on each: the decisive moment and what should have
   happened.
4. Winning is the mirror image: `group_by=win_factor`, and `get_deal_verdict`
   on the largest won deals.

## Deals at risk now

1. `pipeline_stats(status=["open"], group_by=risk)`: how much sits at each
   risk level.
2. `search_deals(status=["open"], risk="high", sort_by=amount,
   sort_order=desc)`: which.
3. `get_deal_verdict` (risks and next steps) and `get_deal_activity` (days
   since the customer last replied) for the ones that matter.
4. Their calls: pass the deal ids as `opportunity_id` to `search_calls` or
   `call_stats`.

## Leads going cold

1. `lead_stats(lead_outcome=["none"], filters={no_activity_for_days: 7})`: how
   many open leads have had no touch for a week.
2. `search_contacts(lead_outcome=["none"], filters={no_activity_for_days: 7},
   sort_by=created_at)`: which, with their owners.
3. `get_contact_activity(contact_id)`: last touch and last reply per channel.

## Speed to lead misses

1. `speed_to_lead_stats(created P, group_by=owner)`: median hours, SLA met %,
   missed and never reached per owner.
2. `search_speed_to_lead(verdict=["missed","not_reached"], created P,
   sort_by=hours_since_lead, sort_order=desc)`: who waited longest.
3. `get_speed_to_lead(contact_id)`: every channel's first touch against the
   SLA for one lead.

## One customer's story

`get_contact` (who, their deals) → `get_contact_timeline` (everything that
happened, newest first) → `get_lead_verdict` / `get_deal_verdict` (why it went
the way it did) → `get_call_grading` / `get_email` for the moments that
decided it.

## A trend

Pick the bucket from the length of the period: a day up to about 45 days, a
week beyond. The bucket names differ per tool:

| Tool | Buckets |
|---|---|
| `call_stats`, `meeting_stats`, `message_stats` | `day`, `week`, `month` |
| `lead_stats` | `created_day`, `created_week`, `created_month` |
| `pipeline_stats` | `created_day` … `closed_month` |
| `speed_to_lead_stats` | `created_week`, `created_month` (no day) |
| `ticket_stats` | `week`, `month` |
| `cost_timeseries` | `bucket=hour\|day\|month` |

Rows come back oldest first. A bucket with no records (a weekend) is
**missing**, not a row of zeros: fill the gaps before drawing a line.

## Support after the sale

1. `ticket_stats(P, group_by=issue_group)`: what goes wrong most.
2. `ticket_stats(P, group_by=responsibility)`: whose side it was on.
3. `search_tickets(filters={needs_action: true})` then `get_ticket`: what is
   still waiting on us.
