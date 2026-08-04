---
name: revenue-impact
description: Attach revenue estimates to UX issues and conversion problems. Use when the user wants to know what a friction point or conversion drop is costing in dollars.
user-invocable: false
---

# Revenue Impact

Estimate the revenue cost of UX issues — put a dollar figure on rage clicks, conversion drops, and error spikes.

## When This Runs

Invoked automatically by `frustration-hunter`, `funnel-doctor`, and `error-forensics` when the user asks "what does this cost us?" or "what's the revenue impact?" Not user-invocable directly.

## Workflow

### Step 1: Get the affected user count

From whichever skill invoked this, extract:
- How many users are affected (from `get_opportunity_stats` or `compute_metric`)
- Over what time period

### Step 2: Get the business numbers

Ask the user for:
- **Average order value (AOV)** or average revenue per user
- **Conversion rate** for the affected flow (if not already known)
- **Traffic volume** to the affected page (if not already known)

If they don't know, offer to compute from Fullstory:
```
fullstory:build_metric(query="completed purchases", output_type="single_number")
fullstory:compute_metric(metric_id)
→ also compute total unique users on the checkout page
→ derive: conversion rate = completed / total
```

For AOV: Fullstory doesn't track revenue natively (unless it's sent as a custom event). If revenue data isn't in Fullstory, ask the user for AOV. Don't guess.

### Step 3: Calculate

Basic formula:
```
Revenue lost = affected users × conversion rate × AOV
```

Example:
- 412 users rage-clicked "Apply Promo" on checkout
- Checkout conversion rate: 20%
- AOV: $85

"Of those 412 users, if the rage click caused even half of them to abandon — that's potentially 41 lost purchases (412 × 20% × 50% abandon rate). At $85 AOV, that's ~$3,500 in the last 7 days, or $182K annualized."

Be explicit about assumptions. Use ranges: "$3,000–5,000" not "$3,472."

### Step 4: Present with context

Always show:
1. **The UX issue** (what's broken)
2. **User impact** (how many affected)
3. **Revenue estimate** (with assumptions stated)
4. **Time period** (per week, per month, annualized)
5. **Confidence level** (high if you have session evidence, low if you're extrapolating from small samples)

## Guidelines

- Never fabricate AOV or conversion rates. If you don't have the data, say "I need your average order value to estimate the revenue impact."
- Use ranges, not precise numbers. Revenue estimates are directional, not accounting-grade.
- Annualize cautiously. A 7-day sample × 52 is a rough heuristic, not a forecast.
- If the UX issue only affects a subset of users (e.g., Safari users on mobile), narrow the calculation to that subset.
- Frame revenue impact as motivation to fix, not as a promise. "Fixing this could recover ~$3,500/week" not "You're losing $3,500/week."
