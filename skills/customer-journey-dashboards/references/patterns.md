# Patterns

Building blocks to plan from. Mix them to fit the request. Each names the
recipe ([recipes.md](recipes.md)) or catalog section ([metrics.md](metrics.md))
it rests on.

## Views

**Overview.** One screen answering "how are we doing": 6–12 tiles (count,
rate, median, $), each with one line of context (vs the team, the sample
size, the denominator), and 2–4 charts on the trend and the mix. Tiles first,
detail below.

**Team, combined.** The selected people as one: totals from the `*_stats`
tools with the team as a list; deals combined from `group_by=owner` rows (one
owner per call). Recipe: "A team's numbers".

**Comparison (leaderboard).** One row per person, one column per measure, every
column sortable, a pinned team-average row. Best and worst per column marked,
only among people with at least 5 records; thinner cells greyed. A row opens
that person's view.

**One person.** The same tiles with the team value beside each, coloured
better or worse (lower is better for hours and days). Then their worst
examples: calls to review, missed leads, risky deals.

**Funnel.** Leads → reached → reached on time → with a deal → won. Each step
from its own stats call, each with its own denominator.

**Why (drill-down).** A count, then the records behind it, then their
verdicts: lost reasons → own-cause losses → `get_deal_verdict`. Recipes: "Why
deals are lost", "What to coach one agent on".

**Trend.** One measure over time. Buckets: a day up to about 45 days, a week
beyond. Sorted by date, gaps filled. Recipe: "A trend".

**Coaching page.** A separate page where AI writes coaching for one person
from their data, opened from the dashboard with the person and dates in the
link. Needs a page that can call both the tools and the model; a stored-query
dashboard usually cannot. See the worked example.

## Filters

- **Period**: a select of named periods (Today, Yesterday, Last 7 days, This
  week, Last week, This month, Last month, Last 30 days, This quarter), plus
  Custom with two dates passed as they are. Default from the plan.
- **People**: chosen once and kept across tabs. Empty selection = an empty
  state and no calls.
- One period for the whole page unless the user asks otherwise. A tile that
  ignores it (open pipeline *now*) says so.

## Navigation

- Tabs per area, sub-tabs for detail. Only the open tab loads its data.
- The link remembers the tab, so a shared link opens where it was sent from.
- Anything clickable (a person, a bar, a row) opens the deeper view.

## Linking two pages

Pass context in the link, never through storage: the person, the dates, the
team. Keep the format to letters, digits and `. _ ~ -`. The worked example
has a tested format.
