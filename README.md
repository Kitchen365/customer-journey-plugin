# Customer Journey plugin

The Customer Journey connector (`https://cj.kitchen365.ai/mcp`) and the
know-how to use it well.

| Skill | What it teaches Claude |
|---|---|
| `customer-journey-dashboards` | how to build any dashboard on CJ data: read the account, ask through the picker, plan, build on live data, verify, hand over. Carries the metric catalog (tool, field, formula, denominator), the rules for combining numbers across tools and people, recipes, design rules, a verify checklist and one worked example |

Facts about a single tool live in the CJ server's own descriptions and
instructions, so every client gets them without this plugin. This skill holds
only what spans several tools, and how to build.

## Install

**claude.ai**, as a plugin (checked 2026-10-27):

1. Zip this repository with its manifest at the root, leaving git out:

   ```
   zip -r ../customer-journey-plugin.zip . -x '.git/*'
   ```
2. Settings → Customize → Plugins → **+ Add** → upload the zip. It installs
   the skill and the `cj` connector together.
3. Its **Connectors** tab → `cj` → sign in with your Automation Hub email and
   password.
4. **Publish to org** gives it to the whole team.

A skill alone also uploads under Settings → Customize → Skills (zip its
folder), with the connector added under Connectors by URL.

**Claude Code / Cowork**: this repository is its own marketplace,
`kitchen365-cj` (`.claude-plugin/marketplace.json`, the plugin at its root).

```
/plugin marketplace add <this repository's URL, or a local path to it>
/plugin install customer-journey@kitchen365-cj
```

The connector is added on install. Sign in once: `/mcp` → `cj` →
authenticate.

A connector added on claude.ai also shows in Claude Code for the same
account. Installing the plugin as well lists every tool twice: keep one.

## Changing a skill

Skills name tools by their bare name (`pipeline_stats`), so they work under
any prefix. Renaming a CJ tool breaks them without any error: search this
folder for the old name in the same change.

A metric added to `skills/customer-journey-dashboards/references/metrics.md` is run
against a real account first, and marked `live` with the date.

Bump `version` in `.claude-plugin/plugin.json`, rebuild the zip and upload it
again: an installed plugin does not update itself.
