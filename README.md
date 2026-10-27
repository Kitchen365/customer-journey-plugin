# Customer Journey plugin for Claude

Read-only access to your Kitchen365 Customer Journey account (leads, deals,
calls, meetings, messages, support tickets, AI cost), and a skill that builds
live dashboards on it.

Installing gives access to nothing by itself: the connector asks each person
to sign in with their own Automation Hub account.

## What's inside

| Part | What it does |
|---|---|
| `cj` connector | the Customer Journey tools, read-only |
| `customer-journey-dashboards` skill | builds any dashboard on CJ data: reads the account, asks what it needs through a multiple-choice picker, proposes a plan, builds on live data, checks the numbers, hands over |

The skill loads on its own when a request fits ("build me a sales
dashboard"). No slash command is needed.

## Install in Claude Code

1. Add the marketplace:

   ```
   /plugin marketplace add https://github.com/Kitchen365/customer-journey-plugin
   ```

2. Install the plugin:

   ```
   /plugin install customer-journey@kitchen365-cj
   ```

   Or run `/plugin`, open **Discover**, pick `customer-journey`, **Install**.
   Restart Claude Code if the tools do not show.

3. Sign in: run `/mcp`, choose `cj`, **Authenticate**. In the browser, sign in
   with your Automation Hub email and password and click **Allow**.

The sign-in lasts about 6 days; after that, `/mcp` and authenticate again.

## Install on claude.ai

1. Settings → Customize → Plugins → **+ Add** → **Add marketplace**.
2. URL:

   ```
   https://github.com/Kitchen365/customer-journey-plugin
   ```

   Leave **Sync automatically** on, then **Sync**.
3. Install `customer-journey` from the marketplace.
4. The plugin's **Connectors** tab → `cj` → **Connect**, and sign in with your
   Automation Hub email and password.
5. Optional: **Publish to org** gives it to your whole team.

## Updating

**Claude Code:**

```
/plugin marketplace update kitchen365-cj
```

then update the plugin from `/plugin` → **Installed**.

**claude.ai:** nothing to do with **Sync automatically** on: a push to this
repository reaches it.

## Good to know

- If the Customer Journey connector was already added on claude.ai, it also
  shows in Claude Code for the same account. With the plugin as well, every
  tool is listed twice: keep one.
- A tool missing from the list was not granted to your account.

## For maintainers

- Facts about a single tool belong in the CJ server's own tool descriptions,
  where every client sees them. The skill holds only what spans several tools,
  and how to build.
- The skill names tools by their bare name (`pipeline_stats`). Renaming a CJ
  tool breaks it without any error: search this repository for the old name in
  the same change.
- A metric added to `skills/customer-journey-dashboards/references/metrics.md`
  is run against a real account first, and marked `live` with the date.
- Every release: bump `version` in `.claude-plugin/plugin.json`.
