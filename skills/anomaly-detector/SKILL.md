---
name: anomaly-detector
description: Proactive anomaly detection — scan metrics for statistical outliers, sudden changes, and unexpected patterns. Use when the user wants to know if something unusual happened, when a metric changed unexpectedly, or for automated health monitoring.
---

# Anomaly Detector

Scan your product data for things that don't look right — sudden spikes, unexpected drops, metrics that broke from their normal pattern.

## When to Use

- "Did anything unusual happen this week?"
- "Alert me if conversion drops below its normal range"
- "Scan all our key metrics for anomalies"
- "Why did page views spike on Tuesday?"
- "Is the error rate abnormally high right now?"
- "Check if any funnel step changed unexpectedly"

## Mental Model

An anomaly is a data point that deviates significantly from the expected pattern. Three types:

1. **Spike**: Sudden increase (errors, traffic, rage clicks)
2. **Drop**: Sudden decrease (conversion, engagement, revenue)
3. **Break**: The pattern itself changed — e.g., a metric that was cyclical suddenly went flat, or a metric that grew steadily now oscillates

## Workflow

### Step 1: Choose what to scan

If the user has specific metrics in mind, scan those. If they say "scan everything," pick a default set:

- **Page views** (traffic health)
- **Errors** (console errors, network failures)
- **Conversion** (key funnel completion)
- **Frustrations** (rage clicks, dead clicks from `get_opportunities`)

Ask: "I'll scan page views, errors, conversion, and frustrations over the last 14 days. That OK?"

### Step 2: Build trend metrics

For each metric, build a trend over a wide time window (14-30 days):
```
fullstory:build_metric(query="page views", output_type="trend")
fullstory:compute_metric(metric_id, time_range="last_30_days")
```

### Step 3: Detect anomalies

For each trend, look for:

**Spikes/drops** (>50% change from previous period's average):
- "Page views: 12,340 daily average, but Tuesday hit 31,200 (+153%) — possible anomaly unless there was a campaign or launch."

**Breaks** (pattern change):
- "Checkout conversion was stable at 20-22% for 3 weeks, then dropped to 14% on July 15 and hasn't recovered."

**Zero events** (metric that had data, now doesn't):
- "Rage clicks on /settings dropped to zero on July 20. Was a fix deployed? Or did tracking break?"

### Step 4: Investigate anomalies

For each anomaly flagged:

1. Check if it correlates with a known event (deploy, campaign, holiday). Check annotations with `get_opportunities` — new frustration signals often coincide with metric anomalies.
2. Narrow the time window around the anomaly point
3. Build a `top_n` version of the metric grouped by dimension (page, device, browser) to see if the anomaly is concentrated or widespread
4. Pull sessions from the anomaly period using `get_sessions(metric_id)` to see what users experienced

### Step 5: Classify

| Anomaly | Classification | Action |
|---------|---------------|--------|
| Error spike on July 15, co-occurs with deploy | **Deploy regression** | Run `deploy-radar` and `error-forensics` |
| Page view spike on Tuesday, no errors | **Traffic event** | Likely a campaign or viral post — not a problem |
| Conversion drop, no deploy, no error change | **UX change** | Run `funnel-doctor` to find the drop-off step |
| Rage clicks doubled, no deploy | **Gradual degradation** | Run `frustration-hunter` to find the broken element |

### Step 6: Report

```
## Anomaly Scan — Last 14 Days

### Found 2 anomalies

1. 🔴 Checkout conversion dropped 14% → 8% on July 15
   - 8 days of sustained lower conversion — not a one-day glitch
   - Co-occurs with v2.3.0 deploy (per annotation)
   - No error spike, no new frustration signals
   → Recommend: run funnel-doctor on checkout flow

2. 🟡 Page views spiked +153% on Tuesday Aug 3
   - Single-day spike, returned to normal Wednesday
   - No error or frustration change
   → Likely external traffic event — not a concern

### 8 other metrics scanned — all normal ✅
```

## Guidelines

- "Anomaly" doesn't mean "problem." A traffic spike from a successful launch is an anomaly, but it's good news. Classify, don't just flag.
- One-day blips are usually noise. Focus on sustained changes (3+ consecutive data points outside the normal range).
- Always check for an annotation or deploy that explains the anomaly. "Conversion dropped on July 15 — that's the same day as the checkout deploy."
- Weekday/weekend patterns are normal. Don't flag "Sunday traffic is lower" as an anomaly.
- If you don't have enough historical data (<7 days of trend), say so: "I need at least 14 days of data for reliable anomaly detection. Right now I can only spot very large swings."
