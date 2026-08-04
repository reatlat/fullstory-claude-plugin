---
name: predictive-alerts
description: Proactive alerting based on trend analysis — predict when a metric will cross a threshold and alert before it happens. Use when setting up early warning systems, monitoring for regressions, or creating proactive alert rules.
---

# Predictive Alerts

Set up proactive alerts — warn the user before a metric crosses a critical threshold, not after.

## When to Use

- "Alert me if conversion is about to drop below 15%"
- "Warn me if errors are trending toward our SLO limit"
- "Tell me when rage clicks are on track to double this month"
- "Set up a watch on the new checkout flow — alert if anything changes"
- "What should I be worried about tomorrow?"

## Mental Model

Reactive monitoring tells you what already happened. Predictive alerting tells you what's *about to* happen based on the trend.

If error rate is 0.8% today, 1.2% yesterday, and 1.6% the day before — you're on track to hit your 3% SLO limit in 3 days. Alert now, not when you breach it.

## Workflow

### Step 1: Define the alert

What metric, what threshold, what direction?

```
Alert: Checkout conversion drops below 15%
Current: 21%
Trend: Declining -0.5pp per week
Predicted threshold breach: ~12 weeks (if trend holds)
→ No immediate alert needed, but set a watch for <18% as early warning
```

### Step 2: Analyze the trend

Build a trend metric and assess the trajectory:
```
fullstory:build_metric(query="checkout conversion rate", output_type="trend")
fullstory:compute_metric(metric_id, time_range="last_90_days")
```

Analyze the trend:
- **Direction**: Up, down, or flat?
- **Velocity**: How fast is it changing? (per day, per week)
- **Acceleration**: Is the change speeding up or slowing down?
- **Volatility**: How much does it bounce around? (noisy trends are harder to predict)

### Step 3: Predict threshold breach

Simple linear projection:
```
Current value: 21%
Weekly change: -0.5pp
Threshold: 15%
Weeks to breach: (21 - 15) / 0.5 = 12 weeks
```

Caveat heavily: "This is a simple linear projection based on the last 4 weeks. Trends change. This is an early warning, not a forecast."

### Step 4: Check for acceleration

Is the decline speeding up?
```
Last week: -0.3pp
This week: -0.5pp
Week before: -0.2pp
→ The decline is accelerating (from -0.2 to -0.5). If it continues accelerating, the breach could happen in 6-8 weeks, not 12.
```

### Step 5: Assess confidence

| Trend Pattern | Confidence | Action |
|--------------|-----------|--------|
| Steady decline, low volatility | High confidence in projection | Alert with specific timeframe |
| Declining but volatile (bouncing) | Medium confidence | Alert as "watch," not "predict" |
| Recent sharp drop (2-3 data points) | Low confidence — could be noise | Investigate before alerting |
| Flat, no trend | No prediction needed | Monitor only |

### Step 6: Set the alert

Based on the analysis, define:
- **Watch threshold**: Early warning (e.g., <18% — triggers investigation)
- **Alert threshold**: Action required (e.g., <15% — triggers incident response)
- **Check frequency**: How often to re-evaluate? (daily, weekly)

"If checkout conversion hits 18%, run `funnel-doctor` on the checkout flow. If it hits 15%, alert the on-call engineer."

### Step 7: Report

```
## Predictive Alerts — Current Watches

### 🔴 ACTIVE: Error Rate Approaching SLO
- Current: 2.1%
- SLO limit: 3.0%
- Trend: +0.15pp per week (accelerating)
- Predicted breach: ~6 weeks (was 12 weeks last check — it's getting worse faster)
- Confidence: High (steady trend, low volatility)
- Action: run `error-forensics` now — don't wait for the breach

### 🟡 WATCH: Checkout Conversion Declining
- Current: 21%
- Alert threshold: 15%
- Trend: -0.5pp per week (steady)
- Predicted reach: 12 weeks at current rate
- Confidence: Medium (moderate volatility)
- Action: monitor weekly. At 18%, run `funnel-doctor`.

### 🟢 HEALTHY: Rage Click Rate Under Control
- Current: 3.2% of clicks
- Alert threshold: 5%
- Trend: Declining (-0.1pp per week)
- No breach predicted

### New This Week
- Mobile conversion gap widening (desktop 22%, mobile 11% — from 12% last month)
- Not at alert threshold yet, but worth investigating
```

## Alert Configuration Template

| Component | Example |
|-----------|---------|
| Metric | Checkout conversion rate |
| Current value | 21% |
| Alert threshold | <15% |
| Watch threshold | <18% |
| Trend | -0.5pp/week |
| Predicted breach | 12 weeks |
| Confidence | Medium |
| Action on watch | Run `funnel-doctor` |
| Action on alert | Page on-call, create incident |
| Check frequency | Weekly |

## Guidelines

- This is trend projection, not machine learning. Be explicit: "I'm projecting from the recent trend. This is directionally useful, not mathematically precise."
- Acceleration matters more than velocity. A metric declining at a steady rate is predictable. A metric declining faster each week needs immediate attention.
- A single data point breaking the threshold is not an alert — it might be noise. Require 2-3 consecutive points outside the watch threshold before triggering.
- Alerts are only useful if they have a defined action. Don't create alerts for "FYI" metrics — every alert should trigger a specific investigation or response.
- Re-baseline after major changes. If you deploy a fix for the declining conversion, the old trend is invalid. Reset the projection.
- For critical business metrics (revenue, checkout), set both watch AND alert thresholds. For secondary metrics, a single watch threshold is enough.