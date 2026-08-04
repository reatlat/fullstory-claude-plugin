---
name: benchmark-analyzer
description: Establish performance baselines and benchmarks — compare current metrics to historical averages, set targets, and track progress over time. Use when setting KPIs, establishing baselines, or measuring against goals.
---

# Benchmark Analyzer

Establish baselines, set targets, and track progress — turn raw metrics into benchmarks you can measure against.

## When to Use

- "What's our baseline conversion rate?"
- "Set a target for checkout completion — and track us against it"
- "How does this quarter compare to Q2?"
- "Are we on track to hit our retention target?"
- "What's a good rage-click rate? Are we above or below average?"
- "Benchmark our error rate against industry standards"

## Mental Model

A metric without context is just a number. A benchmark gives it context:
- **Internal baseline**: What's normal for us? (12-month average, last quarter's average)
- **Target**: Where do we want to be? (goal set by the team, industry standard)
- **Comparison**: How do we compare? (to ourselves over time, to peers if data available)

## Workflow

### Step 1: Define what to benchmark

Ask the user:
- "What metrics matter most?" (conversion, retention, error rate, etc.)
- "What time period should be the baseline?" (last quarter, last 12 months, since launch)
- "Do you have specific targets, or do you want me to suggest them?"

### Step 2: Compute the baseline

For each metric, compute over the baseline period:
```
fullstory:build_metric(query="checkout conversion rate", output_type="single_number")
fullstory:compute_metric(metric_id, time_range="last_12_months")
→ Baseline: 19.8% (12-month average)
```

Also compute the standard deviation or range:
```
fullstory:build_metric(query="checkout conversion rate", output_type="trend")
fullstory:compute_metric(metric_id, time_range="last_12_months")
→ Range: 16% - 24%, typical weekly variance: ±3pp
```

### Step 3: Compare current to baseline

```
Current (last 30 days): 21.2%
Baseline (12-month avg): 19.8%
→ +1.4pp above baseline (+7%)
→ Within normal range (baseline range: 16-24%) ✅
```

### Step 4: Set targets

Based on the baseline, suggest targets:
- **Conservative** (achievable): 21-22% (slightly above baseline)
- **Ambitious** (stretch): 23-24% (top of historical range)
- **Moonshot** (breakthrough): 25%+ (above historical max)

Ask: "Where do you want to set the target?"

### Step 5: Track progress

For each target, build a comparison:
```
Progress toward Q3 target (22% checkout conversion):
Current: 21.2% (96% of target)
On track: ✅ (if trend continues, should hit 22% by end of Q3)
```

### Step 6: Industry context (with caveats)

Fullstory doesn't provide industry benchmarks. But you can provide general context from public research (with appropriate caveats):

```
## Conversion Rate Benchmark — E-Commerce Checkout

Your rate: 21%
Industry average (public data): 15-25%
→ You're in the top half, but not exceptional

Top quartile: >25%
→ If you hit your ambitious target (24%), you'd approach top quartile

⚠️ Industry benchmarks are directional, not precise. Different products, different audiences. Use as context, not comparison.
```

### Step 7: Report

```
## Benchmark Report — Q3 2026

### Checkout Conversion
- Baseline (12-month): 19.8% (range: 16-24%)
- Current (Jul): 21.2% (+1.4pp above baseline)
- Q3 Target: 22%
- Progress: 96% of target, on track ✅
- Industry context: 15-25% for e-commerce (directional)

### Error Rate
- Baseline: 0.9% (range: 0.5-2.1%)
- Current: 0.8% (-0.1pp below baseline)
- Q3 Target: <1.0%
- Progress: Exceeding target ✅

### Rage Clicks
- Baseline: 1,400/month (range: 1,100-2,200)
- Current (Jul): 1,850 (↑32% above baseline) 🔴
- Q3 Target: <1,500/month
- Progress: Off track — promo code issue is driving the spike

### Summary
- 2 of 3 metrics on or exceeding targets
- 1 metric off track (rage clicks) — fix in progress
```

## Baseline Heuristics by Metric Type

| Metric | Typical Baseline Period | Healthy Range |
|--------|------------------------|---------------|
| Conversion rate | 12 months (covers seasonality) | Within ±15% of baseline |
| Error rate | 3 months (faster-moving) | Below 1% for SaaS, below 2% for content |
| Rage clicks | 12 months (spiky) | Below 5% of total clicks |
| Page views | 12 months (seasonal) | Within ±20% of baseline for same month last year |
| Retention | 6 months (cohort-based) | Day 7 >30%, Day 30 >15% for consumer apps |
| Session duration | 3 months | Depends on product — benchmark against yourself |

## Guidelines

- Baselines need enough data. <3 months of data is too noisy. Use at least 6 months, ideally 12.
- Seasonality matters. "Conversion is up vs last month" is meaningless if last month was December (holiday retail spike). Compare to same month last year.
- Targets should be achievable. A target 2x above historical max is demotivating. Suggest a range: conservative → ambitious → moonshot.
- Industry benchmarks are directional. "The average e-commerce conversion rate is 15-25% (per Baymard Institute, 2024)" — always cite the source and caveat the comparison.
- Don't benchmark rage clicks or dead clicks against industry — there's no public data. Benchmark against your own baseline.
- Re-baseline annually. If you've improved the product, the old baseline is stale. "Baseline updated to 12-month average ending Q3 2026."