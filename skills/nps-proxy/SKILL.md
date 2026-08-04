---
name: nps-proxy
description: Behavioral proxy for user satisfaction — use frustration signals, engagement patterns, and conversion behavior as a real-time NPS alternative. Use when the user wants to measure satisfaction but doesn't have survey data in Fullstory.
---

# NPS Proxy

Measure user satisfaction from behavior when you don't have survey data — frustration signals, engagement patterns, and conversion behavior as a real-time proxy for how users feel.

## When to Use

- "What's our user satisfaction looking like?"
- "We don't run NPS surveys — can Fullstory give us a satisfaction signal?"
- "Which users are likely promoters vs detractors based on behavior?"
- "Did satisfaction improve after the redesign?"
- "Show me our 'frustration score' trend over time"

## Mental Model

NPS (Net Promoter Score) asks users: "How likely are you to recommend us?" Fullstory doesn't do surveys. But you can build a behavioral proxy from three signals:

1. **Success signals** (promoters): Completed key tasks, returned multiple times, used advanced features, high engagement
2. **Frustration signals** (detractors): Rage clicks, dead clicks, errors, form abandonment, one-and-done sessions
3. **Engagement signals** (passives): Consistent but shallow usage, moderate session frequency, no strong positive or negative signals

The ratio of success to frustration signals gives you a directional satisfaction measure. It's not a survey. It's a behavioral proxy. Treat it accordingly.

## Workflow

### Step 1: Define success and frustration signals

Work with the user to define what "happy" and "unhappy" look like in their product:

**Success signals** (any 2+ = likely satisfied):
- Completed a purchase, signup, or key conversion
- Used the product 3+ times in 30 days
- Session duration >5 minutes (engaged, not bouncing)
- Used advanced features (beyond basics)
- Returned after first session (retained)
- No frustration events in any session

**Frustration signals** (any 1 = likely unsatisfied):
- Rage clicked any element
- Dead clicked any element
- Hit a console error or network failure
- Abandoned a form mid-completion
- Bounced after 1 page view (<10 seconds)
- Visited help/support pages repeatedly

### Step 2: Build satisfaction segments

```
fullstory:build_segment("users with at least 2 success signals and 0 frustration signals in last 30 days") → promoters
fullstory:build_segment("users with at least 1 frustration signal in last 30 days") → detractors
fullstory:build_segment("users with neither strong success nor frustration signals in last 30 days") → passives
```

### Step 3: Compute the proxy score

```
fullstory:build_metric(query="unique users", output_type="single_number")
→ compute for promoters segment → 4,200
→ compute for detractors segment → 1,800
→ compute for passives segment → 6,400

Behavioral Proxy Score = % Promoters - % Detractors
= (4,200/12,400) - (1,800/12,400)
= 33.9% - 14.5%
= +19.4
```

### Step 4: Present with caveats

```
## Behavioral Satisfaction Proxy — Last 30 Days

### Signal Distribution
- Promoters (2+ success, 0 frustration): 4,200 (33.9%) ✅
- Passives (neither strong positive nor negative): 6,400 (51.6%) 
- Detractors (1+ frustration): 1,800 (14.5%) 🔴

### Proxy Score: +19.4
(A positive score means more promoters than detractors. The theoretical range is -100 to +100.)

### What This Means
About 1 in 3 users show strong satisfaction signals — they're completing tasks, returning, and not hitting frustration. About 1 in 7 users show frustration — they're rage-clicking, hitting errors, or abandoning.

### Top Frustration Drivers
1. Rage clicks on "Apply Promo": 412 users (the biggest detractor driver)
2. Checkout form abandonment: 340 users
3. Console errors on /dashboard: 89 users

### Trend
- This month: +19.4
- Last month: +17.2
- Improvement: +2.2 points 🟢
→ Frustration is decreasing, satisfaction trending up

### ⚠️ Important Caveat
This is a behavioral proxy, NOT an NPS survey. It measures what users DO, not what they SAY. Some frustrated users might still recommend you. Some happy users might not. Use as a directional signal, not a precise metric.
```

### Step 5: Segment the score

Break down by user properties to find where satisfaction lives:

```
By plan:
Enterprise: +42 (high satisfaction — they get value)
Free: +8 (mediocre — some get value, many don't)

By device:
Desktop: +24
Mobile: +8 (mobile users are twice as frustrated)
→ Mobile UX is dragging down overall satisfaction.

By acquisition:
Organic: +28
Paid ads: +4 (paid users are less satisfied — possible expectation mismatch)
```

### Step 6: Track over time

Build a trend of the proxy score:
```
fullstory:build_metric(query="unique users", output_type="trend")
→ compute for promoters, detractors, and passives over time
→ plot: proxy score over the last 6 months
→ annotate key events (deploys, feature launches, campaigns)
```

## Behavioral NPS by Product Type

| Product Type | Strong Success Signal | Strong Frustration Signal |
|-------------|----------------------|--------------------------|
| E-commerce | Completed purchase | Checkout abandonment, payment error |
| SaaS | Used 3+ features, returned weekly | Rage clicks on core workflow, form abandonment |
| Content/Media | Read 3+ articles, returned within 7 days | Bounced after <10s, dead clicks on nav |
| Marketplace | Completed transaction, messaged seller | Search with no results, dead clicks on listings |
| B2B/Enterprise | Multiple team members active, used admin features | Permission errors, feature access denied |

## Guidelines

- Always caveat: "This is a behavioral proxy, not a survey." Repeat it in every report. Don't let anyone mistake this for real NPS.
- The proxy is most useful as a trend. A single month's score (+19.4) is less meaningful than "up from +17.2 last month." Track direction, not absolute value.
- Frustration signals are more reliable than success signals. A rage click definitely means frustration. A completed purchase might mean satisfaction, or it might mean "I needed this and powered through a bad experience." Weight frustration signals higher.
- Don't compare your proxy score to industry NPS benchmarks. An NPS of +40 is "excellent." A behavioral proxy of +40 means something completely different. They're different scales.
- If you DO have survey data (NPS, CSAT, CES), correlate it with the behavioral proxy. You might find that session rage clicks are a leading indicator for NPS drops 2 weeks later — that's much more valuable than the proxy alone.
