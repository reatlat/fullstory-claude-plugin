---
name: frustration-hunter
description: Finds and ranks user frustration signals — rage clicks, dead clicks, form abandonment, error spikes. Use when asking "what's frustrating users", "where are users struggling", or "what's the biggest UX problem right now". Uses Fullstory's opportunity detection, then drills into sessions for root cause.
---

# Frustration Hunter

Find what's frustrating your users, ranked by impact, grounded in real session evidence.

## When to Use

- "What are the biggest frustrations on our site right now?"
- "What's causing friction on the checkout flow for mobile users?"
- "What got worse this week vs last week?"
- "Show me the top rage-clicked elements across our product"
- "Find sessions where users are dead-clicking on the settings page"

## Workflow

### Step 1: Discover opportunities

Call `fullstory:get_opportunities` with a time range (default `last_7_days`). This returns ranked frustration signals — rage clicks, dead clicks, form abandonment, error spikes — each with a user count and page location.

If the user asks about a specific page or flow, scope the opportunities to that area. If they ask "what got worse," use week-over-week comparison (the tool handles this).

### Step 2: Get live stats

For the top 3-5 opportunities, call `fullstory:get_opportunity_stats` on each. This returns:
- Exact user count (not an estimate)
- Device split (mobile vs desktop percentage)
- Page concentration (which pages it happens on)
- Rate-of-change vs prior period (is it getting worse?)
- Frustration rate vs site average

Present the top frustrations as a ranked list:
```
1. Rage clicks on "Apply Promo Code" — 412 users, 73% mobile, /checkout
2. Dead clicks on cart icon — 287 users, Safari-specific, /products
3. Form abandonment at password field — 218 users, 22s avg dwell before leaving
```

### Step 3: Pull sessions for the worst offenders

Call `fullstory:get_sessions_for_opportunity` on the top 1-3 opportunities. Get 3-5 sessions each.

For each session, use the `session-context` agent to read the transcript with a specific task:
- "Did the user's action succeed or fail? What happened after the frustration event?"
- "What element did they rage-click and did anything change on the page after?"
- "Was there an error in the console when this dead click happened?"

### Step 4: Synthesize

Look for patterns across sessions:
- Same element failing across users → likely a bug
- Users clicking non-interactive elements → likely a design issue
- High abandonment after a specific step → likely a UX flow problem
- Errors only on specific browsers/devices → likely a compatibility issue

Present findings with: what's happening, how many users, where it concentrates, what the session evidence shows, and what the likely root cause is.

## Deeper cuts

If the user wants dimensional breakdowns beyond what `get_opportunity_stats` provides (browser version, custom user property, etc.):

1. Build a metric for the specific frustration event: `fullstory:build_metric(query="rage clicks on Apply Promo Code", output_type="top_n")` with the desired grouping dimension
2. Compute it: `fullstory:compute_metric(metric_id)`
3. If scoping to a cohort, attach a segment first: `fullstory:update_metric(metric_id, segment_id)` then compute

## Guidelines

- Default time range: `last_7_days`. Ask before using a different window.
- Start with 3-5 opportunities. If the pattern is clear, stop.
- Always ground findings in session evidence — numbers tell you "what" and "where", sessions tell you "why."
- When presenting, include both the metric (user count, device split) and the qualitative finding (what users actually experienced).
- If an opportunity has a sharp rate-of-change (getting worse fast), flag it prominently.
