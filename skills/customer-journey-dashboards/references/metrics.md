# Metric catalog

One row per business measure: the call that answers it, the field to read, and
how to compute it. **Checked** says how the row was confirmed: `live` = run
against a real account on the date shown, field names and formula matched;
`doc` = from the tool's own description, not yet run.

Notation: `stats(args) → field`. `P` is the period: a named period
(`period=` on calls, meetings, messages, tickets; `created_period=` /
`closed_period=` on leads, speed to lead and deals) or explicit dates
(`start`/`end` on calls, meetings, messages, tickets; `filters={created_from,
created_to}` on leads, speed to lead and deals). Both ends of a date range are
included.

Every stats answer is `{"data": [rows], "pagination": {...}}`. With no
`group_by` there is one row, `group: null`.

## The team

| Measure | Call → field | Notes | Checked |
|---|---|---|---|
| Team members | `find_agents page_size=100` → `agent_id`, `name`, `email`, `active` | 71 people on one account: page through `pagination.total_pages`. Includes inactive people and shared inboxes | live 2026-10-27 |
| One group of people | `find_agents agent_id=[ids]` | never with an empty list | doc |
| Who is signed in | `get_account` → `email`, `role`, `client_id` | | doc |

## Leads (contacts)

| Measure | Call → field | Formula / notes | Checked |
|---|---|---|---|
| New leads | `lead_stats(created P)` → `count` | | live 2026-10-27 |
| Leads won / lost | same → `won`, `lost` | `won`/`lost` are the lead's analysed outcome, not deals | live |
| Lead win rate | same | `won ÷ (won + lost)`. `no_verdict` (open or not analysed yet) stays out | live |
| Open or unanalysed leads | same → `no_verdict` | | live |
| Leads with an open deal | same → `with_open_deal` | | live |
| Leads per owner | `lead_stats(group_by=owner, ...)` | | doc |
| Leads over time | `lead_stats(group_by=created_day\|created_week\|created_month)` | | doc |
| Why leads closed | `lead_stats(group_by=exit_from_status)` or `lead_outcome` | | doc |
| Leads gone cold | `lead_stats(lead_outcome=["none"], filters={no_activity_for_days: 7})` → `count` | pick the days with the user | doc |
| Leads lost through the agent's own handling | `lead_stats(lead_outcome=["lost"], filters={main_cause: "individual"})` | `main_cause`: individual / systemic / customer | doc |

## Speed to lead

| Measure | Call → field | Formula / notes | Checked |
|---|---|---|---|
| Leads measured | `speed_to_lead_stats(created P)` → `count` | | live 2026-10-27 |
| Reached / never reached | same → `reached`, `not_reached` | `reached + not_reached = count` | live |
| On time / late | same → `on_time`, `missed` | `on_time + missed = reached` (196 + 300 = 496) | live |
| **SLA met % of reached leads** | same → `sla_met_pct` | `= on_time ÷ reached`. Leads nobody reached are **not** in it | live |
| SLA met % of all leads | same | `on_time ÷ count`. Compute it; say which one you show | live |
| Typical time to first touch | same → `median_hours` | **median, never `avg_hours`**: touches logged before the lead pull the average down, even negative. Over reached leads only | live |
| How long late leads waited | `speed_to_lead_stats(verdict=["missed"], ...)` → `median_hours` | | doc |
| Per owner / channel / over time | `group_by=owner\|channel\|verdict\|created_week\|created_month` | no `created_day` bucket | doc |
| One channel only | `channel=call\|email\|text` | without it, each lead is judged on its earliest touch on any channel | doc |
| Reached within N minutes | `first_touch_hours_max = N / 60` | 15 minutes = 0.25 | doc |

## Calls

| Measure | Call → field | Formula / notes | Checked |
|---|---|---|---|
| Calls | `call_stats(P)` → `count` | | live 2026-10-27 |
| Outcomes | same → `connected`, `voicemail`, `no_answer`, `no_recording`, `no_outcome` | these five add up to `count`. `no_outcome` = not processed yet | live |
| Connect rate | same | `connected ÷ count`. On this account most calls are `no_recording` (860 of 2,349), so also offer `connected ÷ (count − no_recording − no_outcome)`, "of calls with a recording", and label which | live |
| Talk time | same → `total_duration_seconds` | over the calls matched: add `outcome=["connected"]` for conversation time only | live |
| Average call length | same → `avg_duration_seconds` | | live |
| **Average call score** | `call_stats(outcome=["connected"], P)` → `avg_score` (0–100), `graded` | **always `outcome=["connected"]`**: voicemails are graded too and pull the score around. Sample size = `graded` | live |
| Per agent | `call_stats(group_by=agent, ...)` | `group` = agent_id, `label` = name. 23 rows on one account: page through | live |
| Over time | `group_by=day\|week\|month` | | doc |
| By direction / sentiment / stage | `group_by=direction\|sentiment\|call_stage` | | doc |
| Calls on certain deals | `search_deals` for the ids, then `call_stats(opportunity_id=[ids])` | | doc |

## Call quality (scorecard)

| Measure | Call → field | Formula / notes | Checked |
|---|---|---|---|
| Labels per quality | `call_scorecard(outcome=["connected"], P)` → `calls`, `dimensions[].dimension`, `.graded_calls`, `.labels[].label`, `.count`, `.share_pct` | | live 2026-10-27 |
| Share with the best label | same | the best label's `share_pct` | live |
| Weakest quality | same | lowest best-label share | live |
| Example calls with a bad label | `search_calls(filters={dimension: "<dim>", label: "<label>"}, sort_by=score, sort_order=asc)` | | doc |

**Labels differ per quality and in case.** On one account: empathy and
rapport `Good / False Positive / Bad`; eq_behaviors `good / bad /
false_positive`; personalization `High / Medium / Low / None`; recall
`Strong / Weak / None`; urgency `Appropriate / None / Missing`. Best:
Good, good, High, Strong, Appropriate. Worst: Bad, bad, Low, Weak, Missing.
`None` and `False Positive` are neither. Read the labels each answer returns
rather than assuming this list is complete.

## Meetings

| Measure | Call → field | Formula / notes | Checked |
|---|---|---|---|
| Meetings | `meeting_stats(P)` → `count` | | live 2026-10-27 |
| Average length | same → `avg_duration_seconds` | | live |
| Meeting score | same → `avg_score`, `graded` | often `null` with `graded: 0`: most meetings have no transcript. Show "—" and say why | live |
| Per agent / over time | `group_by=agent\|day\|week\|month` | | doc |
| CRM outcome | `group_by=crm_outcome` | | doc |

## Deals (pipeline)

Two cohorts answer different questions; name the one you use.
**Created in P** (`created_period`): what this period's new deals became.
**Closed in P** (`closed_period`): what was decided in this period.

| Measure | Call → field | Formula / notes | Checked |
|---|---|---|---|
| Deals created | `pipeline_stats(created P)` → `count` | | live 2026-10-27 |
| Pipeline created ($) | same → `total_amount` | | live |
| Win rate | `pipeline_stats(...)` → `win_rate` (%) | `= won ÷ (won + lost) × 100`; open deals stay out. `null` when nothing closed: show "—" | live |
| Won / lost in the period | `pipeline_stats(closed P, status=["won","lost"])` → `won`, `lost`, `win_rate` | | live |
| Revenue won | `pipeline_stats(closed P, status=["won"])` → `total_amount` | | doc |
| Open pipeline now | `pipeline_stats(status=["open"])` → `count`, `total_amount` | no period: what is open today | live |
| Average deal size | → `avg_amount` | | live |
| Days open | → `avg_days_open` | | live |
| Average win probability | → `avg_win_probability` (0–100) | | live |
| Per owner | `pipeline_stats(group_by=owner, ...)` | `group` = agent_id, `label` = name | live |
| By stage / lost reason / risk / design type | `group_by=stage\|lost_reason\|risk\|design_type` | | doc |
| Over time | `group_by=created_week\|closed_month\|...` | | doc |
| Deals gone quiet | `pipeline_stats(status=["open"], filters={no_activity_for_days: 7})` | | doc |
| Deals left silent | `filters={silent_gap_days_min: 5}` | | doc |
| Replied within a day | `filters={response_lag_days_max: 1}` against `response_lag_days_max: 9999` (deals with a measured reply) | | doc |
| Lost through the owner's own handling | `pipeline_stats(closed P, status=["lost"], filters={main_cause: "individual"})` | | doc |

**One owner per call.** `pipeline_stats` and `search_deals` take `owner` as a
single name. For a team, call `group_by=owner` once, keep the team's rows by
`group`, and combine:
`win rate = Σwon ÷ (Σwon + Σlost)`, `avg_amount = Σtotal_amount ÷ Σcount`,
`avg_days_open = Σ(avg_days_open × count) ÷ Σcount`.

## Messages and chats

| Measure | Call → field | Formula / notes | Checked |
|---|---|---|---|
| Emails / texts sent and received | `message_stats(channel=email\|text, P, group_by=direction)` → `count` per `inbound` / `outbound` row | **whole account**: no agent filter exists. Label it so | doc |
| Notes per author | `message_stats(channel=note, group_by=author)` | notes are our team's words | doc |
| Website chats | `message_stats(channel=chat, P)` → `count`, `messages`, `customer_messages`, `answered`, `unanswered` | `answered` = one of our people replied, not just the bot | live 2026-10-27 |
| Chat answer rate | same | `answered ÷ count` | live |

## Support tickets

| Measure | Call → field | Formula / notes | Checked |
|---|---|---|---|
| Tickets, open, resolved, past due | `ticket_stats(P)` → `count`, `open`, `resolved`, `past_due` | | doc |
| Time to resolve | → `median_hours_to_resolve` / `avg_hours_to_resolve` | prefer the median | doc |
| Needing action / fix failed / escalated | → `needs_action`, `fix_failed`, `escalations` | | doc |
| Case quality | → `avg_case_quality` (1–5) | | doc |
| By issue / responsibility / agent | `group_by=issue_group\|responsibility\|agent` | the agent is **Freshdesk's**, not a `find_agents` person | doc |

## AI analysis cost

Periods here are different: `1day`, `1week`, `1month`, `3month`, `overall`, or
`start`/`end` dates in **UTC**. Every answer carries `unit`: `credits`, or
`own_key` (US dollars on the client's own AI keys). Never show a cost without it.

| Measure | Call → field | Checked |
|---|---|---|
| Spend and change | `cost_summary` → `cost_usd`, `prev_cost_usd`, `change_pct` | doc |
| Spend over time | `cost_timeseries(bucket=day)` | doc |
| What costs most | `cost_breakdown(group_by=activity_type\|model)` | doc |
| Credits left | `credit_balance`; a client on its own keys has none | doc |
| Daily cap | `credit_daily_limit` | doc |
| Spend the ledger could not price | `unpriced_usage` | doc |

## Adding a row

Run the call against a real account. Confirm the field names and that the
formula's parts add up (as `on_time + missed = reached` does). Then mark it
`live` with the date.
