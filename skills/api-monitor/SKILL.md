---
name: api-monitor
description: Monitor API call patterns from session data — endpoint latency, error rates, popular endpoints, and API-driven UX issues. Use when investigating backend performance impact on UX, API errors affecting users, or endpoint usage patterns.
---

# API Monitor

Monitor API calls that affect user experience — which endpoints are slow, which are erroring, and how API failures manifest in the UI.

## When to Use

- "Are our API calls causing user-facing slowness?"
- "Which endpoints are returning the most errors?"
- "Did the backend deploy slow down the checkout API?"
- "Show me sessions where /api/orders failed"
- "What's the slowest API call users experience on /dashboard?"
- "Compare API error rates before and after the backend deploy"

## Mental Model

Fullstory captures network requests made by the browser — XHR and Fetch calls. You can see which endpoints users hit, how long they took, and which ones failed. This gives you the *user's* perspective on API performance, which is different from server-side metrics.

A slow API is invisible server-side but very visible to users: spinners, blank pages, rage clicks on submit buttons.

## Workflow

### Step 1: Identify API-heavy pages

Which pages make the most API calls?
```
fullstory:build_metric(
  query="sessions with network activity on /dashboard",
  output_type="trend"
)
```

Or build segments for pages known to be API-heavy: `/checkout`, `/dashboard`, `/search`, `/reports`.

### Step 2: Find API errors from user sessions

```
fullstory:build_metric(
  query="network errors (4xx or 5xx responses) on /checkout",
  output_type="top_n"
)
→ group by endpoint to see which API is failing
```

Present: "504 on /api/orders: 156 users affected (85% of all checkout API errors). 429 on /api/search: 42 users (rate limited)."

### Step 3: Investigate error sessions

For the worst endpoint:
```
fullstory:build_segment("users who received 504 from /api/orders in last 7 days")
fullstory:get_sessions(segment_id, limit=5)
→ session-context agent: "What happened after the 504 error? Did the user retry? See an error message? Abandon the page?"
```

### Step 4: Measure API latency impact

Slow APIs (not errors, but >2s response time) cause UX friction:
```
fullstory:build_metric(
  query="sessions with network request >2000ms on /checkout",
  output_type="single_number"
)
```

Check: do slow API calls correlate with:
- Higher bounce rate?
- More rage clicks?
- Lower conversion?

"Users who experienced a slow /api/orders call (>2s) had a 12% conversion rate vs 24% for users with fast calls. The slow API is literally halving checkout conversion."

### Step 5: Compare before/after deploys

```
fullstory:build_metric(query="/api/orders errors", output_type="trend")
fullstory:compute_metric(metric_id, time_range="last_14_days")
→ look for an inflection point at the backend deploy timestamp
```

### Step 6: Report

```
## API Health — /checkout (last 7 days)

### Error Rates
- /api/orders: 3.2% error rate (504 Gateway Timeout) — 156 users affected 🔴
- /api/payment-methods: 0.1% error rate ✅
- /api/promo/validate: 0.8% error rate (400 Bad Request) 🟡

### Latency
- /api/orders: avg 1.2s, p95 3.8s 🔴
- /api/payment-methods: avg 0.3s ✅
- /api/promo/validate: avg 0.6s ✅

### User Impact
- 504 errors concentrated 7-9 PM ET (peak hours)
- Users who hit a 504: 12% conversion rate (vs 24% for users who didn't)
- Session evidence: users see "Try again" toast, retry once, then abandon

### Correlation
- 504 spike started July 15 — same day as backend deploy v3.1.0

### Recommendations
1. Investigate /api/orders timeout under peak load (P0 — revenue impact)
2. Add retry logic with exponential backoff on the frontend
3. Consider async order submission so 504 doesn't block checkout
```

## API Health Heuristics

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Error rate | <0.5% | 0.5-2% | >2% |
| p95 latency | <1s | 1-3s | >3s |
| Timeout rate | <0.1% | 0.1-1% | >1% |
| Affected users | <10/day | 10-50/day | >50/day |

## Guidelines

- Fullstory captures network requests from the browser, not server-side metrics. You see what the user experiences, not what the server measures. They can differ.
- A 504 error server-side might be a blip. A 504 error user-side means the user saw a failure. Report from the user's perspective.
- Cross-reference API errors with `frustration-hunter` — are users rage-clicking submit buttons after API failures?
- Network capture must be enabled for API monitoring to work. If network requests aren't showing up, check recording block rules (some orgs block API calls for privacy).
- Slow-but-successful API calls (>2s) are silent UX killers. They don't trigger error tracking but they frustrate users and lower conversion.
- After any backend deploy, run this skill against the changed endpoints. API regressions are the #1 deploy-caused issue.
