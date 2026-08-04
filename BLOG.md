# We Built 44 AI Skills for FullStory — Here's What Happened

*By Alex Zappa · August 4, 2026*

The official FullStory Claude Code plugin ships with 3 skills. We shipped 44.

Here's how we did it, what each skill does, and why your product team should care.

---

## The Problem: FullStory Has Too Much Data

FullStory records every session on your site — every click, every scroll, every rage click, every console error. It's a treasure trove of product insights. The problem: you need to know what to look for, how to build segments, how to compute metrics, and how to interpret results. Most product teams have exactly one person who knows how to do this, and they're busy.

FullStory's MCP (Model Context Protocol) server changes that. It lets AI agents — Claude, Cursor, VS Code Copilot — query your FullStory data in natural language. Instead of building metrics in the FullStory UI, you type a question and get an answer.

But the MCP server is a toolbox. It gives you `build_metric` and `get_opportunities` and `session_open`. Knowing *which* tool to use, *when*, and *how to interpret the results* is still a skill gap. That's where skills come in.

## What Skills Are

A skill is a prompt template that teaches Claude a specific workflow. Instead of Claude figuring out from scratch how to investigate a checkout funnel, the `funnel-doctor` skill gives it a battle-tested workflow: define the funnel, build metrics per step, identify the biggest drop-off, pull sessions to explain why.

Skills chain together. You don't say "use the funnel skill" — you say "where are users dropping off in checkout?" and Claude loads the right skill automatically.

## The 44-Skill Collection

We organized 44 skills into five categories:

### Analytics & Measurement (18 skills)
From `quick-stats` ("how many users yesterday?") to `predictive-alerts` ("warn me before conversion drops below 15%"), these skills measure and monitor everything. `general-analysis` handles any quantitative question. `experiment-analyzer` evaluates A/B tests. `retention-analyzer` tracks cohort stickiness. `anomaly-detector` scans for statistical outliers. `lifecycle-analyzer` maps users from new → active → at-risk → churned.

### Investigation & Debugging (13 skills)
From `frustration-hunter` (rage clicks, dead clicks, form abandonment) to `changelog-detective` (find undocumented deploy changes), these skills find and diagnose problems. `error-forensics` traces JS errors to root cause. `session-review` replays user sessions in your terminal. `a11y-analyzer` checks keyboard navigation and WCAG compliance from real sessions. `api-monitor` tracks endpoint latency and errors from the user's perspective.

### Operations & Maintenance (9 skills)
`deploy-radar` validates deploys. `metric-auditor` finds duplicate metrics and stale segments. `privacy-auditor` hunts for PII leaks. `webhook-health` monitors integration delivery. `incident-responder` runs structured incident triage with blast radius estimation and Slack updates.

### Team & Communication (2 skills)
`slack-reporter` formats findings for team channels. `session-playlist` curates annotated playlists of sessions for design reviews or bug reports.

### Onboarding & Learning (2 skills)
`onboarding-tutor` teaches new users the mental model and runs their first query. `query-translator` helps users turn vague questions ("are users happy?") into measurable metrics.

---

## How We Built It

Each skill follows the same structure:

1. **Trigger condition** — What question invokes this skill?
2. **Mental model** — How should Claude think about this problem?
3. **Workflow** — Step-by-step: which tools to call, in what order, what to check
4. **Sample output** — What a real result looks like
5. **Gotchas** — Common mistakes and how to avoid them

We started with the official 3 skills, identified the biggest gaps (frustration analysis, funnels, error debugging), and filled them. Then we kept going — experiments, retention, mobile, accessibility, forms, APIs, webhooks, dashboards, personas, lifecycles, incidents. Every time we thought "someone's going to ask about X," we built a skill for it.

The entire collection is open source (MIT) and installs with one command:

```
/plugin marketplace add https://github.com/reatlat/fullstory-claude-plugin
/plugin install fullstory@fullstory-marketplace
```

---

## What This Enables

With 44 skills loaded, here's what a product team can do in one conversation:

**[See the full transcript →](EXAMPLE.md)**

> **10 skills, one conversation.** Frustration discovery → session investigation → revenue impact → Jira bug report → mobile analysis → Slack summary. All from typing "something's wrong with checkout."

This is what MCP was designed for — not just querying data, but reasoning about it.

---

## What's Next

The skills are open source. If you have FullStory and Claude Code (or Cursor), you can install them today. If you have ideas for more skills, PRs are welcome. If you want to adapt them for a different analytics tool (Amplitude, Mixpanel, PostHog), the workflow patterns are portable.

The official plugin has 3 skills. We shipped 44. Your product team deserves at least a few of them.

---

*[GitHub: reatlat/fullstory-claude-plugin](https://github.com/reatlat/fullstory-claude-plugin) · 44 skills · MIT license*
