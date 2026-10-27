<!--
Worked example, not a template. This spec was built and verified for one
client; its names and ids are replaced by placeholders here. Specific to that
client, and never to be reused as is:
- the timezone it computed date presets in (prefer named periods instead);
- the list of shared inboxes left out of "Active agents";
- the ids in the link example;
- the connector name "CJ MCP" in the capabilities block: use the name the
  connector has in the current client's tool list.
What carries over: the level of detail, the ground rules, the link contract,
the data-source discipline, and the verify-before-handover steps.
-->

# Prompt: CJ Team Performance dashboard + CJ AI Coach

Build two linked artifacts from live Contact Journey data on the **CJ MCP** connector:

1. **CJ AI Coach**: a hand-built HTML artifact where Claude writes coaching for one agent, live, when it opens.
2. **CJ Team Performance**: a Dashboard artifact (start it from the Dashboard type) with 5 tabs. Its Coaching tab opens the AI Coach for the chosen agent, dates and team.

**Build order:** publish the AI Coach first so its URL exists, then build the dashboard and point its Coaching link at that URL.

**Ground rules for both artifacts:**
- Every figure comes from a live CJ tool call for the exact agents and dates selected. Never estimate, never hard-code numbers, and never add up rows yourself when a `*_stats` tool can count on the server.
- Pass `user_question` on every call.
- The account timezone is `<account timezone>`. Compute date presets in it.

---

## Part A — CJ AI Coach (hand-built artifact)

### Why a separate page
A Dashboard artifact cannot call Claude and cannot embed another page, because iframes are dropped. A hand-built artifact can do both through runtime capabilities.

### Capabilities (declare on publish)
```json
{
  "mcp": { "servers": [ { "server": "CJ MCP", "tools": [
    "find_agents", "speed_to_lead_stats", "search_speed_to_lead", "lead_stats", "call_stats",
    "call_scorecard", "pipeline_stats", "search_calls", "search_contacts", "search_deals",
    "get_call_grading", "get_lead_verdict", "get_deal_verdict" ] } ] },
  "sample": {},
  "downloads": true
}
```

### Link contract (how the dashboard passes context)
The URL hash is a set of `.`-joined tokens. Only letters, digits and `. _ ~ -` survive in an artifact hash.

| Token | Meaning |
|---|---|
| `a<agentId>` | Agent to open, e.g. `a1001` |
| `s<YYYY-MM-DD>` | Start date |
| `e<YYYY-MM-DD>` | End date |
| `t<id>-<id>-…` | Agent ids in the dashboard team |
| `n<Team~Name>` | Team name, with spaces written as `~` |

Example: `#a1001.s2026-09-24.e2026-10-23.t1001-1002.nMy~Team`

### Layout
- **Header:**
  - eyebrow "Contact Journey · coaching" and the agent's name as the title;
  - controls: Agent dropdown, Period (Last 7 days, Last 30 days [default], This month, Last month, This quarter, Custom with two date inputs), **Write coaching** (becomes **Rewrite** after a run) and **Download PDF**;
  - under the controls: "My team · N agents" when a team came in the link.
- **Agent dropdown:**
  - with a team in the link, it lists only those people (`find_agents agent_id=[…]`), marking inactive ones;
  - without a team, it lists active agents.
  - There is no "show all agents" option.
- **Status bar:** a progress bar, the current step ("Reading data from Contact Journey…", "Claude is writing 6 parts… 3 of 6 ready") and a **Stop** button.
- **Intro cards before the first run:** pick an agent; Claude writes the coaching; runs on your own account.
- **Headline:** one sentence written by Claude.
- **Real tabs, one part on screen at a time:**
  - Summary · Speed to lead · Lead follow-up · Calls & discovery · Deal follow-up · Outcomes & lost-deal lessons.
  - Each tab shows a dot: pulsing while being written, green when ready, red if it failed. A failed part gets a "Try this part again" button.
- **Summary tab:** two columns, **What went well** (✓ items with a small area label) and **Where to improve** (numbered, a "Priority" tag on high ones, and a "How to improve" box on each).
- **Section tabs, in this order:**
  1. a 2–3 sentence brief;
  2. a strip of key figures straight from CJ (not written by Claude);
  3. What went well / What to improve, with How to improve on each;
  4. **Examples from this period:** 2–3 cards, each with the contact or deal name, "What happened" and "Next time".
- **Footer:** "Written by Claude from live Contact Journey data on <time>. The figures come straight from Contact Journey; the wording and examples are Claude's." List any data that could not be read.
- **Do not show** a "Data Claude used" panel.

### Data gathered on open (all parallel, with the viewer's connector)
Use `CF = {created_from, created_to}` and `CL = {closed_from, closed_to}`.

- **Speed to lead:**
  - `speed_to_lead_stats owner=[name] filters=CF`, plus the same with `verdict=["missed"]` to get how long late leads waited (median);
  - an account-wide `speed_to_lead_stats filters=CF`, used for calibration only;
  - `search_speed_to_lead owner=[name] verdict=["missed","not_reached"]` (6 rows) and `verdict=["on_time"]` (3 rows), for examples.
- **Lead follow-up:**
  - `lead_stats owner=[name] filters=CF`;
  - open leads gone cold: `lead_outcome=["none"]` with `no_activity_for_days: 7`;
  - leads lost through the agent's own handling: `lead_outcome=["lost"]` with `main_cause: "individual"`;
  - `search_contacts` for 4 cold leads, 3 own-cause lost leads (plus `get_lead_verdict` for each: headline, decisive moment, what went wrong, what went well, what should have happened) and 2 won leads.
  - Drop email and phone fields before anything is sent to Claude.
- **Calls:**
  - `call_stats agent=[name]` for all calls and for `outcome=["connected"]`, plus account-wide versions for calibration;
  - `call_scorecard outcome=["connected"]`;
  - `search_calls` for the 5 latest connected calls over 2 minutes (`filters={duration_min:120}`), plus `get_call_grading` for each: grader notes, scorecard labels, suggested next steps, discovery coverage and missed items, agent talk %.
- **Deal follow-up:** `pipeline_stats owner=name` for:
  - deals created in the range (`filters=CF`), and those with `silent_gap_days_min: 5`;
  - `response_lag_days_max: 1` against `9999` (replied within a day, out of deals with a measured reply);
  - open deals now, and open deals with `no_activity_for_days: 7`;
  - plus `search_deals` for 4 silent-gap deals (longest silence first) and 4 stalled open deals (largest first).
- **Outcomes:**
  - deals created in range (won, lost, win rate, amount);
  - lost in range (`filters=CL`), and lost in range with `main_cause: "individual"`;
  - `search_deals` for 3 own-cause lost deals (plus `get_deal_verdict`: the agent's share of the cause, what they did, the customer side) and the 2 largest won deals.
- **Error handling:**
  - an access error (needs reauth, not connected, not allowed, blocked) stops the run with clear "how to fix" text;
  - any other failure leaves that item empty and is listed in the footer.

### Claude calls
- **Six `sample.json` calls run in parallel:** one for Summary and one per section.
- **Settings:** `modelTier: "default"`, cached for 1 hour for identical data; Rewrite sets `refresh: true`.
- **Data per call:** each section call gets only its own section's data, plus the agent, the period and the calibration figures. The summary call gets everything except the example lists.
- **Rules in every prompt:**
  - Plain, clean sentences a manager can read aloud in a 1:1. Refer to the agent by first name. No jargon, markdown or emojis.
  - Never compare the agent with other people or the team, and never mention the team. Account figures are for calibration only and are never cited.
  - Every point rests on a number or a named example and includes that number. Use only names, dates and facts that are in the data.
  - "How" means concrete actions for next week (what, when, how often). Each "text" and "how" is at most two sentences.
  - On time = first touch inside the response window; late = after it; never contacted = no touch.
- **Section output:**
  ```json
  {"brief":"","went_well":[{"text":""}],"improve":[{"text":"","how":""}],"examples":[{"contact":"","what_happened":"","better":""}]}
  ```
  1–3 items in each list and 2–3 examples. If fewer than 5 leads, calls or deals sit behind a part, the brief says so in one sentence and the lists stay empty.
- **Summary output:**
  ```json
  {"headline":"","went_well":[{"area":"","text":""}],"improve":[{"area":"","text":"","how":"","priority":"high|medium"}]}
  ```
  2–4 items in each list, ordered by impact on winning leads.
- **Error copy:** branch on the sample error code (not granted, rate limited, invalid JSON, refused, and so on) and never retry automatically.

### Download PDF
- Use jsPDF 2.5.1 from cdnjs (`jspdf/2.5.1/jspdf.umd.min.js`) and save the file with `downloads.save`.
- Filename: `coaching-<agent-slug>-<start>-to-<end>.pdf`.
- Contents:
  - title, period and team;
  - the headline;
  - the Summary lists;
  - each section's brief, figures, lists with How to improve, and examples;
  - a footer line.
- Replace characters the PDF's standard font can't show (✓, arrows) with plain ones.

### Design
- A calm, clean reading page in a single column, max about 1120px wide.
- Faces: Bricolage Grotesque for display, Public Sans for body text, IBM Plex Mono for labels.
- Colours: cool neutrals with a teal accent, plus green and amber for went well / improve. Light and dark themes are both defined as tokens.
- Works at phone width.

---

## Part B — CJ Team Performance (Dashboard artifact)

### Structure
- **Header:**
  - title;
  - a pill showing the team ("My team · 5", which jumps to tab 1) and a pill showing the date range;
  - "Data as of <time>", **Refresh** and **Download**.
  - **Download** saves the current tab as a styled HTML file through the `downloads` capability. Before cloning the tab, convert the theme colours on charts and inline styles into real colour values so the file looks right on its own. The viewer opens the file and prints it to PDF; `window.print()` does not work inside the dashboard.
- **Global filter** (on tabs 2–4): a date range select with Today, Yesterday, Last 7 days, This week, Last week, This month, Last month, Last 30 days [default], This quarter, and Custom (both ends inclusive). There is no "compare to previous period".
- **Tabs and sub-tabs:**
  - Five numbered tabs; inside them, sub-tabs as a pill row.
  - Only the active tab is drawn, and sub-tab content loads only when that sub-tab is opened.
  - The link remembers the tab and sub-tab, e.g. `#performance-calls`.

### Tab 1 · Your team (the dashboard opens here, with nobody preselected)
- **Building a team:**
  - type a team name;
  - pick from "Start from" chips: "Active agents" (all active agents except the account's shared inboxes, e.g. `<Company> Admin`, `<Company> Marketing`, Customer Service, Design Team, Project Coordinator), "Active + shared inboxes", and any team created this session;
  - search, a "Show inactive" toggle, Select all shown and Clear;
  - agents appear as a grid of selectable cards (initials, name, email).
- **Applying it:** "N selected", then **Create team & view →**, which applies the team and jumps to tab 2.
- **Without a team:** tabs 2–5 show "Create or pick a team first" with a button back to this tab.
- **Note:** teams created here last until the page reloads.

### Tab 2 · Team performance (the selected team combined)
- **Overview:**
  - an insight strip of 3 cards: **Top performer** (highest average connected-call score with at least 5 graded calls; clicking the name opens tab 4), **Needs coaching** (lowest score, with its 2 weakest scorecard qualities as tags) and **Data gaps** (calls with no recording, calls not processed, meetings without a transcript);
  - 11 tiles, each with a serif number and one line of context: Calls · Connect rate · Avg call score (connected) · Talk time · Meetings · New leads · Median speed-to-lead · SLA met · Pipeline created · Win rate (won ÷ closed) · Open deals;
  - charts: calls by outcome over time (stacked bars), a speed-to-lead donut, a deal-status donut, and the average connected-call score as a line.
- **Calls & quality:** stacked calls by outcome, an outcome-mix donut, a connect-rate line, and scorecard-quality bars (share of calls with the best label; red under 50%).
- **Leads & speed:** new leads by verdict over time, a lead-outcome donut and a speed-to-lead donut.
- **Pipeline:** a deal-status donut, open pipeline by stage (amount · count) and the top 8 lost reasons.
- **Meetings & messages:** meetings over time, and emails and texts sent vs received (labelled "whole account").

### Tab 3 · Agent comparison
- **Leaderboard:**
  - columns: Agent | Calls | Connected | Connect % | Avg score | Graded | Talk time | Meetings | Meeting score | Leads | Won | Lost | Win % | Pipeline | Median speed-to-lead | SLA %;
  - every column sorts; there's a filter box; the header and first column stay in place when scrolling; a pinned "Team avg" row sits at the bottom;
  - green marks the best and red the worst value in each column, among agents with at least 5 data points; cells resting on fewer than 5 are greyed;
  - **Export CSV** saves through `downloads`, falling back to the clipboard;
  - clicking a row opens tab 4.
- **Calls & quality:** calls vs connected, average score (with call counts and a dashed team line), connect rate and talk time.
- **Leads & pipeline:** leads stacked by verdict, pipeline $, median speed-to-lead and SLA % (dashed team lines).
- **Scorecard:** a heatmap of agents by quality (% best label) and a radar (the team plus up to 5 agents).
- All per-agent bars are clickable and open tab 4.

### Tab 4 · Agent performance (one agent)
- **Choosing the agent:** a dropdown plus name chips from the team, and an **Open coaching →** button that goes to tab 5.
- **Overview:** 10 tiles, each with the team value under it and a green or red edge (higher or lower than the team, respecting measures where lower is better), plus calls by outcome over time and an outcome-mix donut.
- **Calls:** the score trend against the team (dashed), plus "Calls to review": up to 5 connected calls with the worst label on their weakest quality.
- **Leads & deals:** speed-to-lead and lead-outcome donuts, open deals by stage, a table of missed speed-to-lead leads, and a table of high-risk open deals.

### Tab 5 · Coaching (launcher only)
- **Content:**
  - an agent dropdown;
  - one card: "Written by Claude", "Coaching for <agent>", a short description of the five parts, and chips reading "Opens in a new tab · Takes about a minute · Uses your own Claude usage";
  - one button, **Open <first name>'s coaching ↗**, a normal link with `target="_blank"` to the AI Coach URL. Its hash carries `a<agentId>`, `s<start>`, `e<end>`, `t<team ids joined by ->` and `n<team name with ~>`.
- **Not here:** no coaching data, coaching data sources or coaching sub-tabs live on the dashboard.

### Data sources (live queries on CJ MCP)
Dates are `start`/`end` for calls, meetings and messages; `filters:{created_from, created_to}` for leads, speed to lead and pipeline. The bucket is a day when the range is 45 days or less, otherwise a week (lead_stats uses `created_day` / `created_week`).

- **Team:**
  - `find_agents` (page_size 100) for the full list, and `find_agents agent_id=[ids]` for the selected team.
- **Calls:**
  - `call_stats agent=[names]`: totals, totals with `outcome=["connected"]`, `group_by=agent` (all and connected), and `group_by=<bucket>` (all and connected).
  - Quality scores always use connected calls only.
- **Meetings:**
  - `meeting_stats`: totals, by agent, and by bucket.
- **Speed to lead:**
  - `speed_to_lead_stats owner=[names]`: totals and `group_by=owner`.
  - Show the median, never the average.
- **Leads:**
  - `lead_stats owner=[names]`: totals, by owner, and by created bucket.
- **Pipeline:**
  - `pipeline_stats group_by=owner` for all owners. It filters by only one owner, so a calculation sums the selected agents: win rate = won ÷ (won + lost), and days open is weighted by deal count.
  - Stage and lost-reason views use loaders that call `pipeline_stats owner=<name> group_by=stage|lost_reason` for each owner.
- **Messages:**
  - `message_stats channel=email|text group_by=direction`, account-wide.
- **Scorecards:**
  - `call_scorecard outcome=connected` for the team, then one call per agent (in a loader).
  - Best labels: Good, good, High, Strong, Appropriate. Worst labels: Bad, bad, Low, Weak, Missing.
- **Agent tab:**
  - `call_stats` for the one agent by bucket (all and connected);
  - `call_scorecard` to find the weakest quality, then `search_calls filters={dimension, label:<worst>} sort_by=score asc page_size=5`;
  - `search_speed_to_lead verdict=["missed"]`;
  - `pipeline_stats owner status=open group_by=stage`;
  - `search_deals owner status=open risk=high`.
- **Empty lists:** never send an empty agent or owner list, because it returns everyone. Show the empty state instead.

### Data rules
- Show the sample size next to every average ("61.8 · 48 calls").
- Show "—" for no data, never 0.
- Every number, chart and table is marked to its source or calculation. Shares and rates are calculations.

### Design
- Clean and dense: cards with 1px borders, serif for big numbers and titles, sans for everything else, tabular numerals, sentence case, very little text.
- Theme tokens only, so light and dark both work. Works at laptop and tablet width.
- Charts:
  - bars at most 32px wide with rounded ends;
  - 2px lines with a soft gradient area under them;
  - dashed lines for the team;
  - donuts with the total in the centre and a side legend;
  - horizontal bars with a faint track;
  - tooltips that stay inside the card.
- States: skeletons while loading, "Failed to load" with a small red dot on failure, and 50% opacity while refreshing.

### Known limits
- Agent roles aren't in the CJ team list, so show email instead.
- Calls have no web link, so show the call ID.
- Dashboard teams are not saved. They travel to the AI Coach in the link.
- AI Coach wording changes on each Rewrite; the figures and the facts Claude can draw on stay the same for the same agent and dates.
- Both artifacts are private until shared, and viewers need CJ MCP connected. The AI Coach also uses the viewer's own Claude usage.

---

## Part C — Verify end to end before concluding

Do not say the work is finished until both artifacts have been checked end to end. Report what passed, what failed and what could not be checked.

### Dashboard
1. Run every stored data source once at its default values: the default team and Last 30 days. Fix any call that fails, and re-run it.
2. Read the dashboard's check result after each page change. Fix anything the code confirms, such as a script error, NaN, a blank chart, or overlapping or cut-off labels.
3. Walk the flow:
   - create a team on Your team;
   - open each tab and each sub-tab;
   - change the date range;
   - click a leaderboard row to open Agent performance, and confirm that agent's tiles, charts and tables fill in;
   - go to Coaching and confirm the button's link carries `a<agentId>`, `s<start>`, `e<end>`, `t<team ids>` and `n<team name>`.
4. Spot-check 3–4 numbers against a direct CJ call: team calls, connect rate, SLA met % and one agent's win rate. They must match.
5. Confirm Download saves the current tab as a file, and Export CSV saves the leaderboard. If downloads aren't available in the dashboard, confirm the clipboard fallback and say so.

### AI Coach
1. Open the link from the dashboard's Coaching tab. Confirm:
   - the dropdown lists only the team;
   - the agent and dates are preselected;
   - "My team · N agents" shows.
2. Confirm every CJ call the page makes returns the expected data. Run each tool once with the same arguments here, and check for the fields the page reads.
3. Run one full coaching pass for a real agent. Confirm:
   - all 6 tabs fill in, and each tab shows only its own part;
   - the figure strips match CJ;
   - every example names a contact or deal that is really in that agent's data;
   - no sentence compares the agent with anyone else.
4. Test Rewrite, Stop, a failed part's "Try this part again", and Download PDF. Open the PDF and confirm every section is in it.
5. If a step needs the viewer's approval and can't be run from your session (Claude writing, downloads), say exactly which step wasn't verified.

### Hand over
After verifying:
- **Report briefly:** what each artifact does, what was verified, and anything that could not be.
- **Ask the user to try both artifacts** and say what to improve. Name a few concrete things to look at:
  - whether the numbers match what they expect;
  - whether the coaching tone and examples are useful for 1:1s;
  - whether any tab, chart or measure is missing or not needed;
  - whether the "how to improve" advice fits their team's playbook.
- **Then iterate** on their feedback, one area at a time.
