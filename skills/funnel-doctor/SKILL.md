---
name: funnel-doctor
description: Conversion funnel analysis — find where users drop off, quantify the loss, and diagnose why. Use when asking about conversion rates, funnel abandonment, checkout completion, signup flow performance, or "where are we losing users."
---

# Funnel Doctor

Diagnose conversion funnels — find the leakiest step, quantify the loss, and show you the sessions that explain why users leave.

## When to Use

- "Where are users dropping off in our checkout flow?"
- "What's the conversion rate from signup to first purchase?"
- "Compare the old onboarding flow vs the new one — did we improve completion?"
- "Show me sessions where users abandoned at the payment step"
- "What's the biggest drop-off in our app? Quantify the lost revenue."

## Mental Model

A funnel is a sequence of steps users must complete. The goal is to find the step where the most users leave — not just by absolute count, but by the percentage drop from the previous step.

Example: 10,000 land on /checkout → 8,000 enter shipping → 6,000 reach payment → 2,000 complete purchase. The biggest *absolute* drop is payment (6,000 → 2,000 = 4,000 lost). The biggest *percentage* drop is also payment (67% drop). Both matter — present both.

## Workflow

### Step 1: Define the funnel steps

Clarify what steps the user cares about. Typical funnels:
- **E-commerce**: product page → add to cart → checkout → shipping → payment → confirmation
- **SaaS signup**: landing → signup form → email verification → onboarding → first action
- **App flow**: home → search → results → detail → booking

If the user doesn't specify steps, ask for the start and end point. You can infer the middle steps from page navigation data.

### Step 2: Build a funnel metric

Call `fullstory:build_metric` with:
- `query` describing the funnel steps in order (e.g., "users who viewed /checkout, then reached shipping, then reached payment, then completed purchase")
- `output_type="funnel"` — this tells the builder to create a sequential funnel, not a simple count

If a funnel metric already exists for this flow, search first: `fullstory:get_metric(regex="checkout funnel")`.

### Step 3: Compute and analyze

Call `fullstory:compute_metric(metric_id)`.

Present results as a step-by-step breakdown:
```
Checkout Funnel (last 30 days)
1. /checkout            10,000 users  (100%)
2. Shipping form         8,000 users  (80% — lost 2,000)
3. Payment page          6,000 users  (75% of remaining — lost 2,000)
4. Confirmation          2,000 users  (33% of remaining — lost 4,000) ← BIGGEST DROP
Overall conversion: 20%
```

Call out the step with the biggest absolute drop AND the biggest percentage drop — they're often the same, but not always.

### Step 4: Scope with segments

If the user asks about a specific cohort (mobile users, enterprise, new vs returning), build a segment and attach it:

```
fullstory:build_segment("mobile users") → segment_id
fullstory:update_metric(metric_id, segment_id)
fullstory:compute_metric(metric_id)
```

For A/B comparisons of funnels (old flow vs new, mobile vs desktop), use the `comparisons` skill to pick the right mechanism (dimensionality vs segments).

### Step 5: Investigate the worst step

Once you've identified the leakiest step, find out *why*:

1. Call `fullstory:get_sessions(metric_id=metric_id)` filtered to sessions that reached but didn't complete the problem step
2. Load 3-5 sessions through the `session-context` agent with focused tasks:
   - "What did the user do on the payment page before leaving? Did they encounter an error?"
   - "Was the form validation blocking them? What fields were highlighted?"
   - "Did they rage-click or dead-click anything before abandoning?"

### Step 6: Report

Synthesize everything:
- The funnel numbers (table format)
- Which step is the biggest problem (with both absolute and percentage loss)
- What the session evidence shows about *why* users leave at that step
- If applicable, estimated revenue impact (user count × average order value)

## Guidelines

- Always show both absolute drop (user count) and percentage drop at each step.
- Surface the `metric_url` so the user can verify in the Fullstory UI.
- If a funnel step has a very high drop-off, prioritize investigating that step — don't spend equal time on every step.
- For revenue impact, ask the user for average order value. Don't guess.
- If the user asks "did the new design help?", build two funnel metrics (before/after deploy date) and present side by side.
