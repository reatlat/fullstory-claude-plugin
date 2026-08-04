---
name: weekly-digest
description: Automated weekly summary of what changed in your product — top frustrations, new errors, trends up/down, conversion changes. Use when asking "what changed this week", "give me the weekly recap", or "what's new since last week."
---

# Weekly Digest

A structured weekly summary of what changed in your product: frustrations, errors, conversion trends, and notable shifts.

## When to Use

- "What changed this week vs last week?"
- "Give me a weekly recap of product health"
- "What should I put in the weekly product update?"
- "Anything trending sharply up or down this week?"
- "What new errors appeared this week?"

## Mental Model

The digest compares two time periods — typically "this week" vs "last week" — across four dimensions:
1. **Frustration signals** — rage clicks, dead clicks, form abandonment
2. **Errors** — console errors, network failures, crashes
3. **Conversion** — key funnel metrics
4. **Traffic** — overall page views and session count

The output is a structured report, not a deep investigation. Flag what's worth digging into; the user can then invoke the specific skill (`frustration-hunter`, `error-forensics`, `funnel-doctor`) for the things that matter.

## Workflow

### Step 1: Frustrations

Call `fullstory:get_opportunities(time_range="last_7_days")`. Compare to prior 7 days.

For each opportunity, call `fullstory:get_opportunity_stats`. Flag anything with:
- User count increase >20% week-over-week
- New signal that didn't exist last week
- Frustration rate significantly above site average

Present top 3-5 as a ranked list with week-over-week change.

### Step 2: Errors

Build a metric for errors (console errors, network failures):
```
fullstory:build_metric(
  query="console errors and network failures",
  output_type="trend"
)
fullstory:compute_metric(metric_id, time_range="last_14_days")
```

Flag:
- Errors that started this week (zero last week, non-zero now)
- Errors with >50% increase week-over-week
- Most frequent error overall

### Step 3: Conversion

Pick 2-3 key funnels (checkout, signup, onboarding). If they don't exist in the account, ask the user which funnels matter. Build a trend metric for each:

```
fullstory:compute_metric(funnel_metric_id, time_range="last_14_days")
```

Flag any funnel with >10% drop in completion rate week-over-week.

### Step 4: Traffic

Quick sanity check — is the site getting more or less traffic?
```
fullstory:build_metric(query="page views", output_type="trend")
fullstory:compute_metric(metric_id, time_range="last_14_days")
```

### Step 5: Assemble the digest

Format:
```
## Weekly Digest — August 1–7, 2026

### Frustrations
- Rage clicks on "Apply Promo Code": 412 users (+23% vs last week) 🔴
- Dead clicks on cart icon: 287 users (+12%)
- Form abandonment at password: 218 users (-8%) 🟢

### Errors
- TypeError on /dashboard: NEW — 89 users this week 🔴
- 504 on /api/orders: 156 users (+5%)
- Console error on /settings: resolved (0 users this week) ✅

### Conversion
- Checkout completion: 18% (down from 22% last week) 🔴
- Signup completion: 64% (stable)
- Onboarding completion: 45% (up from 41%) 🟢

### Traffic
- 1.2M page views (+3% week-over-week)
- 89K sessions (+2%)

### Worth investigating
1. Promo code rage clicks are up 23% — run frustration-hunter
2. New TypeError on /dashboard — run error-forensics
3. Checkout conversion dropped 4 points — run funnel-doctor
```

### Step 6: Offer next steps

For each "worth investigating" item, offer to run the relevant deep-dive skill. Let the user pick what matters most.

## Guidelines

- The digest is a status report, not an investigation. Don't pull session transcripts unless the user asks.
- Default comparison: this week vs last week. Ask if they want a different window.
- If a metric doesn't exist yet, build it — but note that next week's digest will be faster since it can reuse the metric.
- Green (🟢) = improving, red (🔴) = worsening, no emoji = stable.
- If everything is stable, say so: "No significant changes this week. Site health is steady."
- Always surface `metric_url` for each metric so the user can verify in the UI.
