---
name: retention-analyzer
description: User retention analysis — N-day retention, cohort stickiness, returning vs new user patterns, and churn signals. Use when asking about retention rates, user stickiness, churn analysis, or "are users coming back?"
---

# Retention Analyzer

Measure user retention — how many users come back, how often, and who's at risk of churning.

## When to Use

- "What's our 7-day retention rate?"
- "How many new users from last month are still active?"
- "Are enterprise users more sticky than free users?"
- "Show me users who haven't returned in 30 days"
- "What's the churn rate for users who signed up in Q2?"
- "Compare retention between users who completed onboarding vs those who didn't"

## Mental Model

Retention is about *returning users*. There are three common lenses:

1. **N-day retention**: What percentage of users who were active on Day 0 came back on Day N? (e.g., Day 1, Day 7, Day 30)
2. **Cohort retention**: Of users who signed up/started in a period, what percentage are still active N days later?
3. **Stickiness**: How many days per week/month do active users engage? (DAU/MAU ratio)

Fullstory doesn't have a native retention report, so you build it from segments and metrics.

## Workflow

### Step 1: Define the retention window

Clarify what the user wants:
- **N-day return**: "Are users active on Day 7 and Day 30?"
- **Ongoing engagement**: "How many sessions per week do active users have?"
- **Churn detection**: "Which users haven't returned in 30+ days?"

### Step 2: Build a retention cohort

For N-day retention from signup:
```
fullstory:build_segment("users with first_seen between July 1 and July 7") → cohort_segment
```

For ongoing activity:
```
fullstory:build_segment("users with last_seen in last 7 days") → active_users
fullstory:build_segment("users with last_seen before last 30 days") → churned_users
```

### Step 3: Measure return rate

Build a metric that counts users who were active in both the cohort window AND the return window:

```
fullstory:build_metric(
  query="users who were active between July 1-7 AND active between Aug 1-7",
  output_type="single_number"
)
```

Compare to the total cohort size to get the retention rate:
"1,200 of 5,000 July Week 1 users were active in August (24% 30-day retention)."

### Step 4: Engagement depth

For stickiness (how often active users engage):
```
fullstory:build_metric(
  query="sessions per user in last 7 days",
  output_type="top_n"
)
→ "45% of active users had 1-2 sessions, 30% had 3-5, 15% had 6-10, 10% had 10+"
```

### Step 5: Segment retention

Using the `comparisons` skill approach, break retention down by cohort:

```
By plan:
Enterprise: 52% 30-day retention
Free:        18% 30-day retention

By onboarding:
Completed onboarding: 41% 30-day retention
Skipped onboarding:    9% 30-day retention ← BIG GAP
```

### Step 6: Churn signals

Identify users at risk of churning:
```
fullstory:build_segment("users with last_seen 14-30 days ago AND total_sessions > 5")
→ these are previously engaged users who've gone quiet
```

For these users, check:
- Did they hit errors or frustrations in their last sessions?
- Did their session frequency drop before they stopped?
- What was the last thing they did?

Use `user-journey` to trace a sample of churned users and look for common patterns.

### Step 7: Present

Format the retention analysis:

```
## Retention — July 2024 Cohort (5,000 new users)

### Return Rates
- Day 1: 68% (3,400)
- Day 7: 41% (2,050)
- Day 30: 24% (1,200)

### By Segment (30-day)
- Enterprise: 52% | Free: 18%
- Completed onboarding: 41% | Skipped onboarding: 9%

### Churn Risk
- 890 users haven't returned in 14-30 days (previously active)
- Common last action: viewed /pricing, did not convert

### Recommendation
Onboarding completion is the biggest lever — users who finish onboarding are 4.5x more likely to stick. The pricing page is where users go before churning.
```

## Guidelines

- Fullstory doesn't have a built-in retention report. Be transparent: "I'm building this from segments and metrics — it's accurate but takes a few extra calls. For dedicated retention analytics, consider a tool like Amplitude or Mixpanel."
- Day 0 = user's first day (from `first_seen`). Day 1 = the next day, Day 7 = a week later, etc.
- For B2B products, retention is typically measured in weeks or months, not days. Adjust the window.
- "Active" usually means "had at least one session in the window." Clarify with the user if they define it differently.
- Segment-level retention differences are often the most actionable insight — "enterprise users retain at 52%, free at 18%" tells you where to focus.
