# Data rules

The connector's own instructions and tool descriptions cover each tool on its
own: stats tools for totals, named periods, medians for speed to lead,
connected calls for call quality, what `sla_met_pct` leaves out. These rules
are what they cannot say: how numbers combine across tools and people, and
what a page that runs again and again must survive.

## Every number

1. **Find the metric in [metrics.md](metrics.md) first**: tool, field, formula,
   denominator.
2. **Rates come from counts, never from averaging percentages.** Three agents
   at 100%, 31% and 0% win rate are not a 44% team: add the counts, then divide.
3. **Every average carries its sample size**: "56.3 · 528 calls". Fewer than 5
   records: shown, marked thin, never ranked.
4. **No data is "—", never 0.** `null` means "nothing to measure".
5. **Say the denominator** of every rate with more than one honest reading.
6. **Name the cohort of a win rate**: deals *created* in the period (what new
   deals became) or *closed* in it (what was decided). On one account, 31% and
   27% in the same 30 days. Keep one across a page.

## Combining across people

- Calls, meetings, leads and speed to lead take a list of people: one call
  for the total, one with `group_by=agent` / `owner` for the rows.
- **Deals take one owner per call.** Call `pipeline_stats group_by=owner` once,
  keep the team's rows by `group` (the agent_id), then combine:
  `win rate = Σwon ÷ (Σwon + Σlost)`,
  `avg_amount = Σtotal_amount ÷ Σcount`,
  `avg_days_open = Σ(avg_days_open × count) ÷ Σcount`.
- Email and text counts have no agent: they are account-wide. Label them so.
- Ticket agents are Freshdesk's, a different id space from `find_agents`.

## Scales and labels

- Call and meeting `score` 0–100; part scores 0–10; deal `win_probability` and
  `engagement` 0–100, except **0–1** in `get_deal_curves`; ticket
  `case_quality` 1–5.
- Scorecard labels differ per quality and in case. Best: Good, good, High,
  Strong, Appropriate. Worst: Bad, bad, Low, Weak, Missing. `None` and
  `False Positive` are neither. Read the labels each answer returns.
- Meetings usually have no transcript, so `graded: 0` and `avg_score: null` are
  normal. "—", and say why.

## A page that runs on its own

- **Never send an empty list.** The connector refuses it; the page shows an
  empty state and makes no call.
- **Always pass a period or a filter on a search**, or a `page_size` of 5 or
  fewer: an unfiltered search over 200 rows returns `ask` instead of rows.
- **`truncated: {shown, of}`** means the result was cut at 30,000 characters.
  Ask for fewer rows, or use a stats tool.
- **Grouped stats are paged**, `page_size` up to 100. Read
  `pagination.total_pages`. Avoid `group_by=contact` on a page.
- **Time groupings list only buckets with records.** Fill the missing ones
  before drawing a line.
- **A refused filter name** comes back with the valid list. Fix it; never drop
  the filter and carry on.
- **Sign-in expiry**: a CJ sign-in lasts about 6 days. An authentication error
  shows "Reconnect Customer Journey", not "no data".
- **Shared inboxes and inactive agents** appear in `find_agents`. Ask which
  inboxes to leave out; keep inactive people for history, not for "the team
  now".

## Shared output

No customer emails or phone numbers unless asked. No recording links ever:
they play for anyone, without signing in, for 24 hours.
