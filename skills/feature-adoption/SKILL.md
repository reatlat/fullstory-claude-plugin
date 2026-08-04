---
name: feature-adoption
description: Track feature adoption — who's using the new feature, how deeply, are they sticking with it, and what's blocking adoption. Use when launching a new feature, measuring feature engagement, or understanding feature usage patterns.
---

# Feature Adoption

Track how users adopt new features — who's using it, how deeply, are they coming back, and what's preventing adoption.

## When to Use

- "How many users have tried the new dashboard?"
- "Are users who tried the new search feature coming back to it?"
- "What's blocking adoption of the new onboarding flow?"
- "Compare feature usage between enterprise and free users"
- "Is the new feature replacing the old one, or are users using both?"
- "Show me sessions where users started using the new feature but stopped"

## Mental Model

Feature adoption is a funnel:
1. **Awareness**: Users who saw the feature (visited the page, saw the announcement)
2. **Trial**: Users who tried it (clicked on it, interacted once)
3. **Adoption**: Users who used it repeatedly (came back, integrated into workflow)
4. **Retention**: Users who are still using it N days later

The drop-off between each stage tells you where the problem is.

## Workflow

### Step 1: Define the feature

Clarify what "using the feature" means:
- A page visit? (e.g., visited /dashboard/new)
- A click event? (e.g., clicked "New Report" button)
- A custom event? (e.g., `feature_search_v2_used`)
- A sequence? (e.g., visited /search → typed query → clicked result)

### Step 2: Measure awareness

How many users encountered the feature?
```
fullstory:build_metric(
  query="users who visited /dashboard/new",
  output_type="single_number"
)
```
If this number is low, the problem is discovery, not the feature itself.

### Step 3: Measure trial

Of those who were aware, how many tried it?
```
fullstory:build_metric(
  query="users who interacted with the new dashboard (clicked any element or spent >10 seconds)",
  output_type="single_number"
)
```

Trial rate = trial users / aware users. Low trial rate = the feature doesn't look compelling or its purpose isn't clear.

### Step 4: Measure adoption (repeat usage)

Of those who tried it, how many came back?
```
fullstory:build_segment("users who used new dashboard at least once")
fullstory:build_metric(query="sessions with new dashboard usage per user in last 30 days", output_type="top_n")
→ "60% used it once, 25% used it 2-3 times, 15% used it 4+ times"
```

The "used once and never returned" group is the most important to investigate.

### Step 5: Segment adoption

Break down by user properties:
```
By plan:
Enterprise: 42% adoption
Free:        8% adoption ← BIG GAP

By onboarding:
Completed onboarding: 35% adoption
Skipped onboarding:    5% adoption
```

A big gap between segments tells you where to focus adoption efforts.

### Step 6: Investigate blockers

For users who tried but didn't adopt:
```
fullstory:build_segment("users who used new dashboard exactly once in last 30 days")
fullstory:get_sessions(segment_id, limit=5)
→ session-context agent: "What did the user do on the new dashboard? Did they encounter errors, confusion, or dead clicks? Why didn't they return?"
```

Look for:
- Errors or crashes during first use
- Rage clicks or dead clicks (confusing UI)
- Users switching back to the old feature immediately after trying the new one
- Users spending very little time (they bounced immediately)

### Step 7: Report

```
## Feature Adoption — New Dashboard (launched Jul 15, 30 days)

### Adoption Funnel
- Aware: 8,200 users (visited /dashboard/new)
- Tried: 3,400 users (41% trial rate)
- Adopted: 1,100 users (32% of trial users, 2+ sessions with feature)
- Retained: 780 users (71% of adopters still using in Week 4)

### Drop-off Analysis
- Biggest drop: Trial → Adoption (68% tried once and never returned)
- These 2,300 users need investigation — run frustration-hunter on /dashboard/new

### By Segment
- Enterprise: 42% adoption | Free: 8% adoption
- Feature isn't reaching free users — consider in-app promotion or gating

### Comparison to Old Dashboard
- Old dashboard still has 5,200 active users
- 1,800 users use both (transitioning slowly)
- 600 users tried the new one, went back exclusively to the old one

### Recommendations
1. Investigate why 68% of trial users don't return (session evidence needed)
2. Promote the new dashboard to free users — they're not discovering it
3. Consider sunset timeline for old dashboard once adoption hits 60%
```

## Feature Lifecycle Stages

| Stage | Metric | Healthy Range |
|-------|--------|--------------|
| Launch week | Trial rate | >25% of users who see it |
| Month 1 | Adoption rate | >20% of trial users return |
| Month 2 | Retention | >50% of adopters still active |
| Month 3 | Cannibalization | New feature usage > old feature usage |
| Month 6 | Default | New feature is the primary path for >80% of users |

## Guidelines

- Always compare feature adoption across user segments — the overall number hides important patterns.
- "Tried once, never returned" is the single most important group to investigate. That's where the UX problems live.
- If the feature is gated (enterprise-only, beta), adjust expectations — adoption will be lower and that's normal.
- Old feature usage is a key metric. If nobody switches, the new feature isn't solving a real problem (or migration is too hard).
- Custom events are the most reliable way to track feature usage. If your team fires a `feature_X_used` event, use that instead of page visits or click patterns.