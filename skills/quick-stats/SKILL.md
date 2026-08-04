---
name: quick-stats
description: Lightning-fast answers to simple numeric questions — "how many users yesterday", "what's the conversion rate", "how many sessions today". Use when the question is a single number, no analysis needed. Skips the full general-analysis workflow.
---

# Quick Stats

One-number answers. No analysis, no session investigation, no validation dance. Just the number, with context.

## When to Use

- "How many users yesterday?"
- "What's the conversion rate this week?"
- "How many sessions today?"
- "How many rage clicks last month?"
- "What's our error count for this week?"

Short questions that expect a single numeric answer.

## Workflow

Everything is one step: build, compute, present.

```
fullstory:build_metric(query="page views yesterday", output_type="single_number")
fullstory:compute_metric(metric_id)
```

Present: "12,340 page views yesterday (Aug 3)."

Add minimal context: "That's +3% vs the daily average last week" — but only if you already have the comparison data. Don't make extra calls to get it.

## When NOT to Use

Don't use this skill if the question is:
- A comparison ("mobile vs desktop") → `comparisons`
- A trend ("is it getting worse?") → `general-analysis` (trend metric)
- A breakdown ("by page" or "by device") → `general-analysis` (top_n metric)
- A "why" question → any of the deep-dive skills

Quick Stats is for "what's the number" — nothing more.

## Guidelines

- One metric, one compute call, one answer. No extra API calls.
- Include the time period in the answer: "12,340 page views yesterday" not "12,340 page views."
- Surface the `metric_url` so the user can verify.
- If the metric already exists from a previous call in the conversation, reuse it. Don't rebuild.
- If the question needs validation (result is zero, looks wrong), don't use Quick Stats — upgrade to `general-analysis` which handles validation.
