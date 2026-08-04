---
name: onboarding-tutor
description: Interactive onboarding guide for new Fullstory MCP users — teaches the mental model, walks through first queries, and explains how to use each skill. Use when a new team member needs to learn how to use Fullstory through Claude.
---

# Onboarding Tutor

Guide new users through Fullstory MCP — teach the mental model, run their first queries, and explain how to use each skill effectively.

## When to Use

- "I'm new to Fullstory MCP — how do I use this?"
- "Teach me how to ask questions about our product data"
- "What can I ask Claude about Fullstory?"
- "Walk me through my first analysis"
- "What skills should I know about?"
- "A new PM joined — onboard them to Fullstory"

## Mental Model

Fullstory MCP turns your product data into a conversation. The key concepts:

1. **Segments** = groups of users ("who")
2. **Metrics** = measurements ("what" and "how much")
3. **Sessions** = evidence ("why")

You don't need to know SQL or analytics. You ask questions in natural language. Claude figures out which tools to use.

## Workflow

### Step 1: Welcome and assess

Ask the new user:
- "What's your role? (PM, engineer, designer, support, exec)"
- "What kind of questions do you want to answer?"
- "Have you used Fullstory's UI before, or is this your first time?"

Tailor the onboarding to their role:
- **PM**: Metrics, funnels, feature adoption, experiments
- **Engineer**: Errors, API monitoring, deploy validation, session debugging
- **Designer**: Session review, mobile analysis, a11y, scroll depth, heatmaps
- **Support**: Session search, customer 360, user journey, bug reproduction
- **Exec**: Dashboards, weekly digests, benchmarks, revenue impact

### Step 2: Run a "hello world" query

Start with something simple and guaranteed to work:

"Let's start with the simplest question: how many users visited your site yesterday?"

Walk through what happens:
1. Claude builds a metric: `fullstory:build_metric(query="page views yesterday", output_type="single_number")`
2. Claude computes it: `fullstory:compute_metric(metric_id)`
3. Claude presents the answer: "12,340 page views yesterday."

Explain that every question follows this pattern: build → compute → present. More complex questions might add: search first → build → segment → compute → validate → investigate sessions.

### Step 3: Teach the skill system

Explain that they don't need to know which skill to invoke — Claude picks automatically:

"Just ask your question naturally. Claude will figure out whether to use `quick-stats` for a simple number, `general-analysis` for a detailed metric, or `frustration-hunter` to find UX problems."

But if they want to invoke a specific skill: "You can also use slash commands like `/fullstory:general-analysis` to force a specific workflow."

### Step 4: Practice with their real data

Now guide them through a question about their actual product:

"What's something you've been wondering about your users? Anything — even if it feels vague."

Take their question and walk through it step by step, explaining what's happening at each stage. Let them see the thinking, not just the answer.

Example conversation:
- User: "I wonder if our new onboarding is working."
- Claude: "Great question. Let me break that down. First, I'll check if there's already a metric for onboarding completion... [searches]. None found. Let me build one. I'm creating a metric that measures 'users who completed onboarding'... [builds]. Now computing... Your onboarding completion rate is 64% over the last 30 days. Want me to compare that to the old onboarding flow, or dig into where users are dropping off?"

### Step 5: Give them a cheat sheet

Share the most useful patterns:

```
How to ask Fullstory questions:

"How many...?"              → gives you a number
"Show me a trend of..."     → gives you a chart description
"Break down... by..."       → gives you a table
"Compare... vs..."          → gives you side-by-side
"What's frustrating...?"    → finds UX problems
"What's breaking...?"       → finds errors and bugs
"Show me sessions where..." → finds specific user recordings
"Trace user X's..."         → follows one user's journey
"What changed this week?"   → gives you a weekly summary
"Did the deploy break...?"  → checks before/after a release
"File a bug for..."         → creates a Jira-ready bug report
```

### Step 6: Give them resources

Point them to:
- **This plugin's README**: Full skill catalog with triggers
- **Fullstory MCP docs**: developer.fullstory.com/mcp
- **Example workflows**: developer.fullstory.com/mcp/workflows
- **FAQ**: developer.fullstory.com/mcp/faq

And remind them: "You can always ask me 'what skills are available?' or 'how do I ask about X?' — I'll guide you."

## Onboarding by Role (5-Minute Version)

### For PMs
```
Try these three questions on your first day:
1. "What's our weekly active user count?"
2. "Show me a trend of checkout conversion over the last 90 days"
3. "What are the biggest frustrations on our product right now?"

That covers: a KPI, a trend, and UX health. Everything else builds from these three.
```

### For Engineers
```
Try these three questions on your first day:
1. "What errors are users hitting in production?"
2. "Find a session where a user hit a JavaScript error and tell me what happened"
3. "Did anything break after the last deploy?"

That covers: error monitoring, debugging, and deploy validation.
```

### For Designers
```
Try these three questions on your first day:
1. "Show me 5 recent sessions on our key page — what do users actually do?"
2. "What elements are users rage-clicking on the checkout page?"
3. "How far do users scroll on the landing page?"

That covers: session review, UX friction, and content visibility.
```

## Guidelines

- The first query should always succeed. Pick something simple and guaranteed to have data.
- Don't explain the entire MCP protocol. They don't need to know about JSON-RPC or tool schemas.
- Let them drive. After the hello-world query, ask "what do YOU want to know?" — don't lecture.
- If they ask something the data can't answer, say so and suggest a proxy. "I can't measure user intent, but I can find users who visited /pricing without converting."
- Celebrate their first insight. "You just discovered that rage clicks are up 23% on the promo code button — that's a real UX problem your team can fix. Nice find."
