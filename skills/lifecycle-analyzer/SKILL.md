---
name: lifecycle-analyzer
description: Analyze the user lifecycle — new, active, at-risk, and churned users. Use when understanding user progression, identifying at-risk users, or measuring lifecycle conversion rates.
---

# Lifecycle Analyzer

Map your users across the lifecycle — new, engaged, at-risk, churned — and understand what moves them from one stage to the next.

## When to Use

- "How many users are at risk of churning?"
- "What percentage of new users become active?"
- "Show me the lifecycle breakdown for Q3"
- "What's the most common path from new → churned?"
- "Which lifecycle stage has the biggest drop-off?"
- "Compare lifecycle between enterprise and free users"

## Mental Model

Every user is in one of these lifecycle stages:

| Stage | Definition | Segment |
|-------|-----------|---------|
| **New** | First session in last 7 days | `first_seen in last 7 days` |
| **Activating** | 2-5 sessions, exploring features | `2-5 sessions, first_seen in last 30 days` |
| **Active** | Regular usage, established patterns | `6+ sessions in last 30 days` |
| **At-Risk** | Declining usage, previously active | `was active (6+ sessions) last month, <3 sessions this month` |
| **Churned** | No activity in 30+ days | `last_seen before 30 days ago` |
| **Resurrected** | Returned after churning | `last_seen before 30 days ago, but active in last 7 days` |

## Workflow

### Step 1: Build lifecycle segments

Build one segment per stage:
```
fullstory:build_segment("users with first_seen in last 7 days") → new_users
fullstory:build_segment("users with 2-5 sessions and first_seen in last 30 days") → activating
fullstory:build_segment("users with 6+ sessions in last 30 days") → active
fullstory:build_segment("users with 6+ sessions last month and <3 sessions this month") → at_risk
fullstory:build_segment("users with last_seen before 30 days ago") → churned
```

### Step 2: Measure lifecycle distribution

For each segment, get the user count:
```
fullstory:build_metric(query="unique users", output_type="single_number")
→ compute for each segment
```

Present the distribution:
```
Lifecycle Distribution (Aug 2026)
New:         2,100 (14%)  🆕
Activating:  3,400 (23%)  🌱
Active:      5,800 (39%)  ✅
At-Risk:     1,900 (13%)  ⚠️
Churned:     1,700 (11%)  💤
Total:      14,900
```

### Step 3: Analyze stage transitions

Track how users move between stages:

**New → Activating:**
```
Of 2,100 new users in July:
- 1,400 became activating (67% activation rate)
- 700 churned immediately (33% — these are the "one-timers")
```

**Activating → Active:**
```
Of 3,400 activating users:
- 2,100 became active (62% conversion to active)
- 800 stayed activating (still exploring)
- 500 became at-risk (lost momentum)
```

**Active → At-Risk (churn signal):**
```
1,900 currently at-risk users:
- 1,200 were active and stopped gradually (declining sessions)
- 700 were active and stopped abruptly (possible: achieved their goal, found alternative, or had a bad experience)
```

### Step 4: Investigate transition blockers

For the biggest drop-off (New → Activating at 67%), investigate why users don't return:
```
fullstory:build_segment("users with 1 session, first_seen in last 30 days")
fullstory:get_sessions(segment_id, limit=10)
→ session-context: "What did this user do in their only session? Did they hit an error, get confused, or just not find what they wanted?"
```

### Step 5: Segment the lifecycle

Break down by user properties:
```
By plan:
Enterprise: 72% active, 8% at-risk, 4% churned
Free:       28% active, 18% at-risk, 18% churned
→ Free users are 4.5x more likely to churn. The free-to-paid conversion gap is the biggest lifecycle lever.
```

### Step 6: Report

```
## Lifecycle Analysis — Q3 2026

### Stage Distribution
- 14,900 total users
- Healthy ratio: 53% of users are active or activating ✅
- Watch: 13% at-risk — if half churn, that's 950 users lost this month

### Key Transitions
- New → Activating: 67% (good, but 700 one-timers/month means 8,400/year lost)
- Activating → Active: 62% (the biggest improvement opportunity — focus onboarding here)
- Active → At-Risk: 13% of active users become at-risk each month (stable rate)

### At-Risk Profile
- 1,900 at-risk users
- 63% were previously daily users → rapid disengagement
- Common last action: visited /settings or /billing → may be an account management issue
- 32% show frustration signals (rage clicks, errors) in their last 2 sessions

### Recommendations
1. Improve activation: users who complete onboarding are 3x more likely to become active. Focus on onboarding flow.
2. At-risk intervention: email users who miss 3+ consecutive days (especially daily users). Offer support.
3. Churn investigation: 700 one-timers/month. Run `session-review` on 10 one-timer sessions to understand what they saw.

### Lifecycle Funnel
New (2,100) → Activating (1,400, 67%) → Active (2,100*, 62%) → At-Risk (1,900, 13%) → Churned (1,700)

*2,100 became active this month, plus 3,700 already active = 5,800 total active
```

## Lifecycle Benchmarks

| Metric | Healthy | Concerning | Critical |
|--------|---------|-----------|----------|
| New → Activation rate | >60% | 40-60% | <40% |
| Activation → Active rate | >50% | 30-50% | <30% |
| Active → At-Risk rate | <10% | 10-20% | >20% |
| At-Risk → Churn rate | <30% | 30-50% | >50% |
| Resurrection rate | >5% | 2-5% | <2% |

## Guidelines

- Lifecycle is a model, not reality. Users don't read your lifecycle definition. Some "churned" users are on vacation. Some "at-risk" users are just busy. Use as a directional tool, not a precise measure.
- The biggest lifecycle lever is usually activation (New → Active). Users who become active in the first 30 days are 3-5x more likely to retain long-term.
- At-risk detection is most useful with an intervention. If you can't act on it (no email, no in-app message), it's just a dashboard widget. Pair with `predictive-alerts` for proactive outreach.
- Segment lifecycle by acquisition channel. Users from paid ads may have different lifecycle patterns than organic users. Use `campaign-tracker` to segment.
- Resurrection is underrated. Users who churn and come back are often more loyal than users who never left. Track it.
