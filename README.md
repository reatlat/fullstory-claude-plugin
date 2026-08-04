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

Twenty-two skills ship with the plugin. Claude loads the right one automatically based on what you ask.

### Analytics & Measurement

| Skill | Trigger | What it does |
|-------|---------|-------------|
| `quick-stats` | "How many users yesterday?" | One-number answers. Build → compute → present. No analysis overhead. |
| `general-analysis` | Any quantitative question | Full workflow: classify intent, search/build, compute, validate, investigate |
| `comparisons` | A vs B questions (auto) | Picks dimensionality vs segments correctly — prevents silent wrong results |
| `funnel-doctor` | "Where are users dropping off?" | Funnel step analysis, drop-off quantification, session evidence for why |
| `weekly-digest` | "What changed this week?" | Structured report: frustrations, errors, conversion, traffic — week-over-week |
| `page-performance` | "Which page has the most errors?" | Page-level health: error rates, device breakdown, navigation patterns |
| `experiment-analyzer` | "Did the variant win?" | A/B test analysis: impact, significance assessment, segment breakdown, side effects |
| `retention-analyzer` | "Are users coming back?" | N-day retention, cohort stickiness, churn signals, engagement depth |
| `anomaly-detector` | "Did anything unusual happen?" | Proactive scan: spikes, drops, pattern breaks across all key metrics |

### Investigation & Debugging

| Skill | Trigger | What it does |
|-------|---------|-------------|
| `frustration-hunter` | "What's frustrating users?" | Rage clicks, dead clicks, form abandonment — ranked, with session evidence |
| `error-forensics` | "What's breaking in production?" | JS errors, network failures, console exceptions — root cause from sessions |
| `session-review` | Session URL or bug report | Open → view → diff → close. Visual replay without a video player |
| `session-search` | "Find sessions for user 84721" | Search by user ID, device, browser, page, custom event |
| `user-journey` | "Trace user@example.com's sessions" | Multi-session timeline, friction points, churn signals |
| `jira-bug-reporter` | "File a bug for this session" | Auto-populated bug reports with session URL, errors, repro steps, severity |

### Operations & Maintenance

| Skill | Trigger | What it does |
|-------|---------|-------------|
| `deploy-radar` | "I just deployed — did anything break?" | Before/after comparison: errors, conversion, frustrations |
| `metric-auditor` | "Clean up our metrics" (auto) | Find duplicates, stale segments, misconfigured objects |
| `cohort-compass` | Cohort analysis (auto) | Build, track, reuse segments across analyses |
| `annotation-ops` | "Mark the deploy on the timeline" (auto) | Create deploy markers, experiment windows, incident tags |
| `revenue-impact` | "What does this cost us?" (auto) | Attach dollar estimates to UX issues and conversion drops |
| `segment-wizard` | "Help me build a segment for..." | Guided interactive cohort building for non-technical users |
| `privacy-auditor` | "Are we capturing PII?" | Audit session capture for PII leaks, verify masking and exclusion rules |

### Agents (isolated context workers)

| Agent | What it does |
|-------|-------------|
| `session-context` | Reads a single session transcript in isolation — keeps main context clean |
| `batch-session-reader` | Reads multiple sessions in parallel — 5x faster for investigation workflows |

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
