---
name: deploy-radar
description: Post-deploy validation — check errors, conversion, and frustrations before and after a deploy timestamp. Use when the user deploys code and wants to know if anything broke.
---

# Deploy Radar

Post-deploy health check: compare key metrics before and after a deploy to catch regressions, new errors, and conversion changes.

## When to Use

- "I just deployed — did anything break?"
- "Check if the 3pm deploy caused any issues"
- "Compare error rates before and after the July 15 deploy"
- "Did the checkout change improve or hurt conversion?"

## Workflow

### Step 1: Define the deploy window

Get the deploy timestamp from the user. If they say "3pm today" or "just deployed", pin to the nearest hour.

Define two time windows:
- **Before**: e.g., 24 hours before deploy (or last 7 days if it was a big release)
- **After**: e.g., from deploy time to now

Ask: "I'll compare the 24 hours before and after the deploy. That OK, or do you want a wider window?"

### Step 2: Check errors

Build an error metric and compute it for both windows:
```
fullstory:build_metric(query="console errors and network failures", output_type="single_number")
fullstory:compute_metric(metric_id, time_range=before_window)
fullstory:compute_metric(metric_id, time_range=after_window)
```

Also check for *new* errors that didn't exist before:
```
fullstory:build_metric(query="console errors", output_type="top_n")
→ compute for after window, compare to before window
→ flag error types that appear only in the after window
```

### Step 3: Check frustrations

Call `fullstory:get_opportunities` for the after window. For each, call `fullstory:get_opportunity_stats`. Check if any opportunity's rate-of-change vs the before window shows a spike.

Quick sanity check: rage click count before vs after. If it doubled, the deploy introduced friction.

### Step 4: Check conversion

Pick 1-2 key funnels (checkout, signup):
```
fullstory:compute_metric(funnel_metric_id, time_range=before_window)
fullstory:compute_metric(funnel_metric_id, time_range=after_window)
```

### Step 5: Report

```
## Deploy Health — v2.3.0 (Aug 4 15:00 UTC)

### Errors
- Before: 12 errors (24h window)
- After: 14 errors (+17%) 🟡
- New: TypeError on /checkout (3 occurrences, did not exist before) 🔴

### Frustrations
- Rage clicks: 87 before → 92 after (+6%) 🟢 (within normal variance)
- Dead clicks: stable
- No new opportunity signals detected 🟢

### Conversion
- Checkout completion: 21% before → 20% after (-5%) 🟡
- Signup completion: 64% → 63% (-2%) 🟢

### Verdict: Deploy is mostly clean. The new TypeError on /checkout is worth investigating — 3 users in 24h, looks like a null check regression. Want me to dig into those sessions?
```

### Step 6: Offer investigation

For any red or yellow flags, offer to run the relevant deep-dive skill (`error-forensics`, `funnel-doctor`, `frustration-hunter`).

## Guidelines

- The narrower the windows, the noisier the data. For low-traffic pages, use at least 48h windows.
- A small change before/after doesn't always mean the deploy caused it — natural variance exists. Flag it, but don't claim causation without session evidence.
- New errors (zero before, non-zero after) are the strongest signal. Prioritize those.
- If the user has annotation-ops enabled, create an annotation for the deploy so future analyses can reference it.
- Always include absolute numbers, not just percentages. "5% increase" from 1 to 1.05 errors means nothing. "5% increase" from 100 to 105 is noise.
