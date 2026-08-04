---
name: dashboard-builder
description: Build and manage recurring dashboard views of key metrics — set up daily/weekly monitoring with saved metrics and segments. Use when the user wants to create a recurring dashboard, set up KPI monitoring, or establish a baseline.
---

# Dashboard Builder

Set up recurring metric dashboards — define key metrics, save them for reuse, and generate daily or weekly snapshots.

## When to Use

- "Set up a product health dashboard"
- "What metrics should I track weekly?"
- "Create a KPI dashboard for the leadership team"
- "I want to check the same metrics every Monday — automate it"
- "What's our north star metric and supporting KPIs?"

## Mental Model

A dashboard is a curated set of metrics you check regularly. It answers "how are we doing?" at a glance. Good dashboards have:

1. **North star metric**: The one number that measures success (e.g., weekly active users, monthly revenue, tasks completed)
2. **Supporting KPIs**: 3-5 metrics that explain the north star (e.g., conversion rate, retention, error rate)
3. **Health indicators**: Quick sanity checks (traffic volume, error count, rage click count)
4. **Trend context**: Week-over-week or month-over-month change

## Workflow

### Step 1: Interview the user

Ask what matters:
- "What's the primary goal of your product? (revenue, engagement, retention, growth)"
- "Who is this dashboard for? (PM, engineering lead, exec, yourself)"
- "How often will you check it? (daily, weekly, monthly)"
- "What would make you panic? (conversion drops 20%, errors spike, traffic crashes)"

### Step 2: Design the dashboard

Based on the answers, propose a dashboard structure:

**For a PM:**
```
North star: Weekly active users
|— New user signups (trend)
|— Feature adoption rate (single_number)
|— User retention (Day 7, Day 30)
|— Top frustrations (from opportunities)
|— Conversion funnel (checkout or activation)
Health checks: error rate, page view volume
```

**For an engineering lead:**
```
North star: Error-free session rate
|— Top errors (by frequency)
|— API error rate (by endpoint)
|— Rage click count (trend)
|— Deploy frequency and success rate
|— Page load performance (slowest pages)
Health checks: traffic volume, session count
```

**For an exec:**
```
North star: Revenue or conversion
|— Conversion rate (trend)
|— Revenue impact of top UX issues
|— New vs returning user split
|— Feature adoption (key features)
|— CSAT proxy (frustration rate)
Health checks: traffic, session count, error rate
```

### Step 3: Build all metrics

Build every metric in the dashboard:
```
fullstory:build_metric(query="weekly active users", output_type="trend") → metric_wau
fullstory:build_metric(query="new user signups", output_type="trend") → metric_signups
fullstory:build_metric(query="error rate", output_type="single_number") → metric_errors
... etc
```

Save all metric IDs. These are now the dashboard. The next time the user wants to check, just recompute these IDs — no rebuilding.

### Step 4: Generate the dashboard

Compute all metrics for the desired time window and present:

```
## Product Health Dashboard — Week of Aug 4, 2026

### North Star: Weekly Active Users
18,420 (+3.2% vs last week) 🟢

### Supporting KPIs
- New signups: 1,240 (+5.1%) 🟢
- Feature adoption (new dashboard): 32% of active users 🟡
- Day 7 retention: 41% (-2pp) 🟡
- Checkout conversion: 21% (-1pp) 🟢

### Health Checks
- Error rate: 0.8% (stable) 🟢
- Rage click count: 1,200 (-8%) 🟢
- Page views: 1.4M (+2%) 🟢

### Top Frustration
Rage clicks on "Apply Promo Code" — 412 users (+23%) 🔴

### Verdict: Healthy week. Promo code issue needs attention.
```

### Step 5: Set up recurring checks

Offer to create a `weekly-digest` configuration or suggest:
"To check this dashboard next week, just say 'dashboard' and I'll recompute these metrics. Everything is saved — no rebuilding needed."

For automation, suggest a cron or scheduled prompt: "Every Monday at 9am, say 'show me the dashboard' to Claude."

## Dashboard Templates

### SaaS Product Health
```
North star: Weekly active users
Supporting: Signup conversion, Day-7 retention, Feature adoption (top 3), Error rate
Health: Page views, Session count, Rage click count
```

### E-Commerce Health
```
North star: Revenue or Orders
Supporting: Checkout conversion, Cart abandonment, AOV, Traffic by channel
Health: Error rate, Page load issues, Dead clicks on product pages
```

### Content Site Health
```
North star: Page views or Subscribers
Supporting: Scroll depth (key pages), Bounce rate, Returning visitor rate, Article completion rate
Health: Error rate, Dead clicks, Broken links (404s)
```

### Mobile App Health
```
North star: Daily active users
Supporting: Session length, Screen flow completion, Crash rate, Feature adoption
Health: API errors, Rage taps, Viewport issues
```

## Guidelines

- Start small. A dashboard with 5-7 metrics is better than one with 20. The user won't look at 20 metrics every week.
- Every metric needs a baseline. On first setup, note: "Baseline established Aug 4. I'll compare to this going forward."
- Explain what each metric means and why it's on the dashboard. Don't just list numbers.
- If a metric is flat/stable, don't narrate it. "Stable" is the note. Save attention for what changed.
- Offer to evolve the dashboard. After 2-3 weeks, ask: "Are these the right metrics? Anything you want to add or drop?"
- Dashboards are per-user, not per-org. If Alice and Bob both say "dashboard," they get different ones based on their roles. Ask first: "Product or engineering dashboard?"
