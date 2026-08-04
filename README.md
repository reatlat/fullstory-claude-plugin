<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/reatlat/fullstory-claude-plugin/main/.github/assets/hero-dark.svg">
    <img alt="Fullstory for Claude" src="https://raw.githubusercontent.com/reatlat/fullstory-claude-plugin/main/.github/assets/hero-light.svg" width="600">
  </picture>
</p>

<p align="center">
  <strong>Talk to your product data.</strong><br>
  Ask Claude anything about your users — drop-offs, rage clicks, conversion funnels — and get answers backed by real sessions.
</p>

<p align="center">
  <a href="#quick-install"><strong>Install</strong></a> ·
  <a href="#what-it-does">What it does</a> ·
  <a href="#skills">Skills</a> ·
  <a href="#prerequisites">Prerequisites</a> ·
  <a href="#troubleshooting">Troubleshooting</a>
</p>

<br>

## Quick Install

One command in Claude Code:

```
/plugin marketplace add https://github.com/reatlat/fullstory-claude-plugin
/plugin install fullstory@fullstory-marketplace
```

Also works in **Cursor** (Marketplace panel, same URL) and **VS Code Copilot** (Chat: Install Plugin from Source).

No local server. No API keys to juggle. OAuth handles auth — you sign in once and it just works.

<br>

## What It Does

Fullstory records every session on your site. This plugin lets Claude *read* that data.

**Before:** Export CSV → pivot table → screenshot → Slack → someone asks "but why?" → repeat.

**After:** Type a question. Get an answer.

| You ask | Claude does |
|----------|------------|
| *"What's the top frustration on our product right now?"* | Builds a rage-click metric across all pages, surfaces the worst offenders, pulls sessions to show you exactly what users are clicking |
| *"How many users abandoned checkout last month, by device?"* | Resolves or builds a funnel metric, attaches segment, computes, returns a table with percentages |
| *"Show me sessions where enterprise users hit errors on settings"* | Builds enterprise segment, finds error sessions, reads each transcript in isolation, synthesizes patterns |
| *"Is our new onboarding flow actually reducing drop-off?"* | Trend metric comparing before/after deploy date, with statistical context |
| *"Find session #abc123 and tell me why the submit button didn't work"* | Opens session, steps through key moments, diffs before/after the click, reports what broke |

It doesn't just answer "how many." It answers "why" — by reading the actual sessions behind the numbers.

<br>

## Skills

Three skills ship with the plugin. Claude loads the right one automatically based on what you ask.

### `general-analysis` — Metrics, segments, and compute

Triggered by any quantitative question. The skill:

1. **Classifies intent** — count? breakdown? trend? comparison?
2. **Searches before building** — checks if someone already built this metric, ranks by popularity
3. **Builds or resolves** — creates new segments/metrics or reuses existing ones
4. **Computes** — runs the metric against your data
5. **Validates** — zero results? anomalies? trend discontinuities? catches them automatically
6. **Investigates** — if the number needs explaining, pulls sessions to find the *why*

Reference docs for [validation rules](skills/general-analysis/references/validation.md) and [session investigation patterns](skills/general-analysis/references/sessions.md).

### `comparisons` — A/B done right (auto-invoked)

Compares anything — mobile vs desktop, enterprise vs free, Chrome vs Safari. The skill knows the hard part: event properties (device, browser, page) use *dimensionality* while user properties (plan, signup date, account) use *segments*. Picking wrong silently produces wrong results. This skill picks right every time.

### `session-review` — Session replay in your terminal

Point Claude at a session URL and it:
- Opens the session, reads the event transcript
- Finds key moments (navigations, clicks, errors, rage clicks)
- Takes visual snapshots at those timestamps
- Diffs before/after to see what changed
- Reports what happened — no video player needed

### Agent: `session-context`

Reads session transcripts in an isolated context window. Keeps your main conversation clean — you can pull 20 sessions without blowing up context.

<br>

## Prerequisites

1. **Fullstory account** with [MCP Beta access](https://www.fullstory.com/blog/fullstory-mcp/)
2. **StoryAI features enabled** — an org admin toggles this in [Settings > Account Management > StoryAI Features](https://app.fullstory.com/ui/org/settings/genai-features)
3. **Claude Code**, **Cursor**, or **VS Code Copilot**

Without StoryAI enabled, the server connects but exposes zero tools. If you see "connected but no tools," that's the fix.

<br>

## How It Works

```
You  →  Claude Code  →  Fullstory MCP Server (api.fullstory.com)  →  Your Fullstory Data
                         ↑ OAuth, no local process
```

The plugin wires up three things:

| File | Role |
|------|------|
| `.mcp.json` | Points Claude at `https://api.fullstory.com/mcp/fullstory` |
| `skills/*/SKILL.md` | Teaches Claude *how* to think about Fullstory data — mental models, workflows, edge cases |
| `agents/*.md` | Isolated context workers for heavy lifting (session transcripts) |

The MCP server exposes tools like `build_metric`, `compute_metric`, `get_sessions`, `session_open`, `session_view`, `session_diff` — you don't memorize these. You ask questions. Claude picks the tools.

<br>

## Troubleshooting

| Problem | Fix |
|---------|-----|
| **"Connected but no tools"** | StoryAI features are off. Admin: enable in Settings > Account Management > StoryAI Features |
| **OAuth fails** | Org might not be in MCP Beta. Contact Fullstory support. |
| **Rate limited** | 3 req/s, 20-request burst. Space out big queries or narrow time ranges. |
| **Zero results that seem wrong** | The `general-analysis` skill auto-validates — it'll broaden filters, expand time range, and cross-check before reporting zero |

Full FAQ: [developer.fullstory.com/mcp/faq](https://developer.fullstory.com/mcp/faq/)

<br>

## Star History

If this is useful, star the repo — it helps other teams discover it.

<a href="https://star-history.com/#reatlat/fullstory-claude-plugin&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=reatlat/fullstory-claude-plugin&type=Date&theme=dark">
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=reatlat/fullstory-claude-plugin&type=Date&theme=light">
    <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=reatlat/fullstory-claude-plugin&type=Date">
  </picture>
</a>

<br>

## License

MIT © [Alex Zappa](https://github.com/reatlat)

Skill content adapted from [fullstorydev/fullstory-skills](https://github.com/fullstorydev/fullstory-skills) (MIT).

<p align="center">
  <sub>Not affiliated with Fullstory. Fullstory is a trademark of Fullstory, Inc.</sub>
</p>
