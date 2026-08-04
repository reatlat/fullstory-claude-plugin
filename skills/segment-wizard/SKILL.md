---
name: segment-wizard
description: Interactive guided segment building for non-technical users — helps construct complex user cohorts with validation and preview. Use when a non-analyst user needs to build a segment but doesn't know how to express it in a query.
---

# Segment Wizard

Guided, step-by-step segment building for non-technical users. Helps PMs, designers, and support build precise user cohorts without knowing the query syntax.

## When to Use

- "I need a segment of users who did X but not Y"
- "Build a segment for our target audience — I'll describe who they are"
- "I want to find users who visited the pricing page but didn't sign up"
- "Create a cohort of power users for me — active, enterprise, non-trial"
- "Help me build a segment — I'm not sure how to express this"

The `segment-wizard` is invoked when the user struggles to express a segment as a natural-language query, or when `general-analysis` detects that a `build_segment` call returned a segment that doesn't match what the user described.

## Workflow

### Step 1: Interview the user

Ask structured questions to narrow down the cohort:

**Time window**:
- "New users this month, or all users regardless of when they joined?"
- "Users active in the last 7 days, or any user who's ever been active?"

**Behavioral criteria**:
- "What did they do? (visit a page, click something, complete an action, see an error)"
- "What did they NOT do? (didn't convert, didn't return, didn't complete onboarding)"

**User properties** (if they have custom properties):
- "What plan are they on? What role? What country?"

**Engagement level**:
- "Power users (many sessions) or casual users (few sessions)?"
- "Have they been active recently?"

### Step 2: Translate to query

Turn the user's answers into a `build_segment` query:

User says:
- "People who visited the pricing page in the last 30 days"
- "But didn't sign up"
- "On enterprise plan"
- "Active in the last 7 days"

Query: "enterprise plan users who visited /pricing in last 30 days, did not visit /signup/confirmation, and were active in last 7 days"

### Step 3: Validate the segment

Build the segment, then verify it matches expectations:

```
fullstory:build_segment(query) → segment_id
fullstory:get_sessions(segment_id, limit=5)
```

Check the 5 sample sessions:
- Do these users actually match the criteria? (visited /pricing, on enterprise plan, etc.)
- Is the segment size reasonable? (not zero, not the entire user base)

Report to the user: "I built a segment matching your criteria. It has 1,200 users. I checked 5 sessions — all visited /pricing, all are on enterprise plans, none signed up. Does that sound right?"

### Step 4: Offer preview stats

Before using the segment in analysis, give the user a quick preview:

```
fullstory:build_metric(query="page views", output_type="single_number")
fullstory:update_metric(metric_id, segment_id)
fullstory:compute_metric(metric_id)
```

"Your segment (1,200 enterprise users who browsed pricing without signing up) had 4,800 page views in the last 30 days — about 4 views per user. Ready to use this segment in analysis?"

### Step 5: Name it

Suggest a clear name: "enterprise-pricing-no-signup-30d". The user can rename it. Good names are self-documenting so other team members can find and reuse the segment.

### Step 6: Hand off

Once the segment is built and validated, hand off to whichever skill the user wants:
- `general-analysis`: "Want me to compute some metrics for this cohort?"
- `frustration-hunter`: "Want to see what frustrates these users?"
- `funnel-doctor`: "Want to see where in the funnel they're dropping off?"

## Common Segment Templates

| Description | Query |
|-------------|-------|
| New users this month | "users with first_seen in last 30 days" |
| Power users | "users with total_sessions > 20" |
| Churned users | "users with last_seen before last 30 days and total_sessions > 5" |
| High-value non-converters | "users who visited /pricing 3+ times in last 30 days but did not visit /signup/confirmation" |
| Mobile-only users | "users with all sessions on mobile devices" |
| Users who saw a specific error | "users with console error 'TypeError: undefined' on /dashboard" |

## Guidelines

- Don't overwhelm the user with questions. Ask 2-3 at a time, build a draft segment, then refine.
- Always validate with sample sessions before presenting results — a misbuilt segment produces wrong numbers.
- If the user's description is impossible to express as a Fullstory segment (e.g., "users who thought about buying but didn't"), say so and suggest a proxy: "We can't measure intent, but we can find users who visited /pricing and /checkout without completing a purchase."
- For complex segments with multiple criteria, build incrementally: first the time window, then the behavior, then the properties.
- If a built segment returns 0 users, don't assume it's wrong — it might genuinely be an empty set. Validate by broadening one criterion: "0 users matched 'enterprise + visited /pricing + from Germany.' Let me check if there are any enterprise users from Germany at all."
