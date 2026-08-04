---
name: cohort-compass
description: Advanced cohort management — build, compare, and track user cohorts over time. Use when working with persistent user groups (enterprise vs free, new vs returning, by country or plan) across multiple analyses.
user-invocable: false
---

# Cohort Compass

Build, manage, and compare user cohorts. Keeps segments consistent across multiple analyses so you don't rebuild the same cohort five times in one conversation.

## When This Runs

Invoked automatically by `general-analysis`, `comparisons`, `funnel-doctor`, and `weekly-digest` when they need cohort-level analysis. Not user-invocable directly — it's a supporting skill that keeps the other skills consistent.

## What Cohorts Are

A **cohort** is a segment that persists across analyses. Examples:
- Enterprise users (plan = enterprise)
- New users (first_seen in last 30 days)
- Power users (total_sessions > 20)
- German users (country = DE)
- Mobile-only users (device = mobile, no desktop sessions)

The key difference from one-off segments: cohorts get reused. You build the "enterprise users" segment once, then attach it to multiple metrics — error rate, conversion, page views, frustration signals — without rebuilding.

## Workflow

### Step 1: Build the cohort

When a skill needs a cohort (e.g., "compare enterprise vs free users"):

```
fullstory:build_segment("users on enterprise plan")
→ returns segment_id
```

Name it clearly in the query so the segment is findable later. "enterprise plan users" not "segment 1".

### Step 2: Track it

Keep a running list of segments built in the conversation:
- `seg_ent`: enterprise users
- `seg_free`: free users
- `seg_new`: users with first_seen in last 30 days

When a later question needs the same cohort, reuse the `segment_id`. Don't rebuild.

### Step 3: Compute per cohort

For each cohort, attach the segment, compute, store the result:
```
fullstory:update_metric(metric_id, segment_id=seg_ent)
fullstory:compute_metric(metric_id) → store result
fullstory:update_metric(metric_id, segment_id=seg_free)
fullstory:compute_metric(metric_id) → store result
```

### Step 4: Compare

Present results side by side:

```
Checkout completion rate (last 30 days)
Enterprise: 31% (3,100 / 10,000)
Free:       18% (5,400 / 30,000)
```

Always show both the percentage AND the raw numbers. A 31% rate with 100 users is different from 31% with 10,000.

### Step 5: Verify cohort membership

If the user questions whether the cohort is correct:
```
fullstory:get_sessions(segment_id=seg_ent, limit=3)
→ spot-check the sessions to confirm they match the cohort criteria
```

## Cohort Freshness

User properties change over time — a user moves from free to enterprise, or from Germany to France. Fullstory resolves user properties to the **last known value**, so segments are always current.

But this means:
- A cohort built today will *not* produce the same results as the same cohort built a month ago (users moved buckets).
- For "snapshot in time" comparisons (what was the conversion rate *at that point in time*), you'd need point-in-time segments, which aren't supported in the current MCP.

If the user needs historical cohort snapshots, tell them honestly that segments use current property values and can't look back in time. Offer to use dimensionality (`top_n` grouped by the property) instead — but note the split-value caveat from the `comparisons` skill.

## Guidelines

- Reuse segment IDs within a conversation. Don't rebuild.
- When presenting cohort comparisons, always show raw counts alongside percentages.
- If the user asks for a cohort that's a slight variation of an existing one, use `fullstory:update_segment` rather than building from scratch.
- For time-based cohorts (new users, users who signed up in Q3), the segment filters by `first_seen` date — make sure the date range is explicit in the query.
- Cohorts are per-conversation, not persistent across sessions. If the user comes back tomorrow, they'll need to rebuild.
