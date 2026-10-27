---
name: customer-journey-dashboards
description: Build a live dashboard, report page or tracker on Kitchen365 Customer Journey (CJ) data, shaped to what this client needs. Use whenever someone asks for a dashboard, scorecard, leaderboard, KPI page, team or agent performance view, pipeline or speed-to-lead view, coaching page, or any page or chart that should show CJ numbers — even when they say it in one line ("build me a sales dashboard").
---

# Customer Journey dashboards

Every client wants a different dashboard, so there is no template here. This
skill is the way to build one: find out what it is for, plan it from the
metric catalog, build it on live data, and prove the numbers before handing
it over.

**Every number follows [references/data-rules.md](references/data-rules.md)**
and comes from the metric catalog,
[references/metrics.md](references/metrics.md): tool, field, formula,
denominator. Read both before planning. Multi-step flows (team views, coaching
evidence, why deals are lost, risk, cold leads, trends) are in
[references/recipes.md](references/recipes.md).

If the CJ tools are not in your list, tell the user to connect Customer
Journey and stop. A missing tool was not granted to this account; a tool
answering "not available yet" has no live data. Leave that area off the page
and say so; never rebuild it from other tools.

## Asking the user: the picker, always

Every question to the user goes through the **multiple-choice picker** (your
ask-the-user or multiple-choice tool). Never ask in prose, never as a bulleted
list, never as "let me know if...". Only a client with no picker asks in chat.

- **2–4 options per question**, each a real answer, the likeliest first and
  marked "(Recommended)". The picker adds "Other" for anything else.
- **Several questions in one picker** when they are independent, so the user
  answers once. Ask up to 4 at a time.
- **Multi-select** when several answers can be true (which areas, which people).
- **Options come from the account**, never invented: real agent names from
  `find_agents`, real stages from `pipeline_stats group_by=stage`, real
  shared-inbox names. Read first, then ask.
- **Ask only what cannot be found out or sensibly defaulted.** One round
  before building is the aim, two at most. A default the user did not choose is
  stated in the plan, where they can change it.
- **Never ask about mechanics** (tool names, page sizes, how data is fetched).
  Ask about the business: who, what decision, which period, which people.

The question bank, with ready options, is in
[references/questions.md](references/questions.md).

## The workflow

### 1. Read the account before asking anything

In parallel: `find_agents page_size=100` (all pages), and one cheap `*_stats`
call per area the request touches, for the last 30 days. This tells you:

- which tools this account was granted (a missing tool = that area is off the
  dashboard, said once in the plan);
- which areas have data (meetings with `graded: 0` cannot have a score tile);
- the people, the inactive ones, and what look like shared inboxes;
- the real values to offer in the picker.

### 2. Ask, through the picker

Usually: who the dashboard is for and what they decide with it; which areas;
which people make up the team (and which shared inboxes to leave out); the
default period. Skip any the request already answered. See questions.md.

### 3. Propose the plan, confirm it with the picker

A short outline in chat: tabs, the tiles and charts on each, the filters, the
default period, and the source of each number (tool and formula from the
catalog). Then one picker: "Build this" (Recommended) / "Change something" /
"Fewer tabs". Patterns to plan from: [references/patterns.md](references/patterns.md).

### 4. Build it on live data

- **Every figure comes from a live tool call** for the exact people and dates
  selected. Never hard-code a number, never estimate, never add up rows a
  `*_stats` tool can count.
- **Start from the platform's dashboard type when there is one**: it stores
  each data source as a live query. A hand-built page that calls the tools
  itself declares the connector under its MCP capability, by the name it has
  **in this client's tool list** (it may be "CJ MCP", "cj" or anything the
  client chose), and lists exactly the tools it calls.
- **Pass `user_question` on every call**: what the page answers, e.g. "Team
  performance dashboard: calls for the selected team, last 30 days".
- Design rules: [references/design.md](references/design.md).

### 5. Verify before saying it is done

Run [references/verify.md](references/verify.md): every data source once, the
whole flow, 3–4 numbers checked against a direct tool call. Report what
passed, what failed and what could not be checked. Never call it finished
untested.

### 6. Hand over, and ask for feedback with the picker

Give the link and two or three lines on what it shows. Then one picker
(multi-select): "Numbers look right" / "Something's missing" / "Something's
not needed" / "A number looks wrong". Iterate on the answer, one area at a
time.

## Things that differ per client: never hard-code them

- **Timezone.** Use named periods; they are read in the account's timezone.
  Ask (picker) only if a date must be computed.
- **Shared inboxes and non-sales users.** Detected from `find_agents`,
  confirmed with the picker.
- **Who counts as the team.** Chosen on the page, or confirmed with the picker.
- **Thresholds** ("stale" after 7 days, "thin" under 5 records). State the
  default in the plan.
- **Granted areas.** Read from the tool list, every time.

## Worked example

[examples/team-performance.md](examples/team-performance.md) is a complete
spec that was built and verified for one client: a five-tab team dashboard
plus a linked AI coaching page. Use it to see the level of detail and care
expected. Do not copy it: its timezone, shared inboxes and tabs were that
client's.
