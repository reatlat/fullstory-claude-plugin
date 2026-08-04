# Fullstory Claude Code Plugin

One-command install to give Claude Code (and Cursor) access to your Fullstory data — behavioral analytics, session replays, funnels, and customer experience insights.

## Quick Install

### Claude Code

```
/plugin marketplace add https://github.com/reatlat/fullstory-claude-plugin
/plugin install fullstory@fullstory-marketplace
```

### Cursor

Install from the Marketplace panel using the same repo URL.

### Manual / OpenSkills

```
npx openskills install reatlat/fullstory-claude-plugin
```

## What You Get

After install, Claude Code can:

- **Query analytics in natural language** — "What are the top frustrations on our product right now?"
- **Build segments and metrics** — "How many users abandoned checkout last month, broken down by device?"
- **Read session replays** — "Find sessions where users rage-clicked on the payment page and tell me what went wrong"
- **Compare cohorts** — "Do enterprise users have more errors than free users?"
- **Validate results** — Automatic zero-result detection, anomaly checks, trend discontinuity detection

### Included Skills

| Skill | Trigger | What it does |
|-------|---------|-------------|
| `general-analysis` | Any quantitative question | Classifies intent, searches/builds metrics, computes results, validates automatically |
| `comparisons` | A vs B questions (auto) | Picks dimensionality vs segments correctly — prevents wrong A/B results |
| `session-review` | Session URL or bug report | Opens sessions, identifies key moments, diffs states, reports findings |

### Included Agent

| Agent | What it does |
|-------|-------------|
| `session-context` | Reads session transcripts in an isolated context — keeps main conversation clean |

## Prerequisites

1. **A Fullstory account with MCP Beta access** — An org admin must enable **Model Context Protocol (MCP)** in [Settings > Account Management > StoryAI Features](https://app.fullstory.com/ui/org/settings/genai-features)
2. **StoryAI features enabled** — Without this, the MCP server connects but shows zero tools
3. **Claude Code** or **Cursor**

## MCP Server

The plugin connects to Fullstory's managed MCP server at `https://api.fullstory.com/mcp/fullstory`. No local process required. Authentication happens via OAuth on first use.

## File Structure

```
fullstory-claude-plugin/
├── .mcp.json                    # MCP server config (HTTP endpoint)
├── plugin.json                  # Root plugin manifest
├── .claude-plugin/
│   ├── plugin.json              # Claude Code plugin manifest
│   └── marketplace.json         # Marketplace listing
├── .cursor-plugin/
│   └── plugin.json              # Cursor plugin manifest
├── agents/
│   └── session-context.md       # Session transcript reader agent
├── skills/
│   ├── general-analysis/
│   │   ├── SKILL.md             # Analytics workflow skill
│   │   └── references/
│   │       ├── validation.md    # Zero/anomaly result validation
│   │       └── sessions.md      # Session investigation patterns
│   ├── comparisons/
│   │   └── SKILL.md             # A/B comparison skill
│   └── session-review/
│       └── SKILL.md             # Session replay review skill
└── README.md
```

## Sharing With Your Team

1. Push this repo to your GitHub org (public or private)
2. Team members run one install command (see Quick Install above)
3. On first use, each person authenticates with their Fullstory account via OAuth

To customize the plugin name/author, edit `plugin.json`, `.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json`.

## Troubleshooting

**"Connected but no tools available"** — StoryAI features are turned off in your Fullstory org. An admin must enable them in Settings > Account Management > StoryAI Features.

**OAuth fails** — Make sure your org is in the MCP Beta program. Contact Fullstory support if you're not enrolled.

**Rate limits** — 3 requests/second with 20-request burst allowance. Space out large queries.

## License

MIT — do whatever you want with this. The Fullstory MCP server and API are Fullstory's proprietary service.

## Credits

Skill content adapted from [fullstorydev/fullstory-skills](https://github.com/fullstorydev/fullstory-skills) (MIT).
