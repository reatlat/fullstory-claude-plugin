---
name: error-forensics
description: Investigate JavaScript errors, console errors, network failures, and crashes in production. Use when debugging user-reported issues, tracking error frequency, finding sessions with specific errors, or determining whether a bug is widespread or isolated.
---

# Error Forensics

Find what's breaking in production — JavaScript errors, network failures, console exceptions — and trace them to root cause using real session evidence.

## When to Use

- "Are users hitting JavaScript errors on the settings page?"
- "Find sessions with 504 errors on /api/orders"
- "What's the most common console error right now?"
- "A user reported the checkout broke — find their session and tell me what went wrong"
- "Is the TypeError on /dashboard affecting many users or just one?"
- "What errors started appearing after the last deploy?"

## Workflow

### Step 1: Discover error patterns

Start broad. Call `fullstory:get_opportunities` to surface error-related frustration signals — these will include console errors, network failures, and renderer errors ranked by user count.

If you're investigating a specific error or page, build a metric instead:
```
fullstory:build_metric(
  query="console errors on /settings",
  output_type="top_n"
)
```

### Step 2: Quantify

Call `fullstory:get_opportunity_stats` or `fullstory:compute_metric` to get:
- How many users are affected
- Device/browser breakdown (is it Safari-only? Mobile-only?)
- Page concentration (which pages is it happening on?)
- Trend (is it getting worse? Did it start after a deploy date?)
- Rate vs site average (is this page abnormally error-prone?)

### Step 3: Find sessions

Call `fullstory:get_sessions_for_opportunity` or `fullstory:get_sessions(metric_id)` to get 3-5 sessions with the error.

### Step 4: Investigate each session

For each session, use the `session-context` agent with a focused task:

```
"Was there a JavaScript error in this session? 
If yes: what was the error message, what page was the user on, 
what action triggered it, and what was the stack trace?"
```

### Step 5: Open the worst sessions

For sessions that look like the canonical case, use `session-review` to:
- Open the session: `fullstory:session_open(session_url)`
- Navigate to the moment of the error
- Use `fullstory:session_view` to see the UI state at error time
- Use `fullstory:session_diff` to compare just before/after the error — what changed (or didn't)?

### Step 6: Report

Present findings:
1. **Error summary**: message, frequency, affected users, device/browser breakdown
2. **Reproduction path**: what the user did before the error (from session transcripts)
3. **Visual evidence**: what the UI looked like at the moment of failure
4. **Severity assessment**: widespread vs isolated, data loss vs cosmetic, getting worse vs stable
5. **Recommended fix**: based on the evidence — is it a null check, a missing API response field, a race condition?

## Error Classification

| Symptom | Likely Cause | Check |
|---------|-------------|-------|
| `TypeError: Cannot read property 'x' of undefined` | Missing null check, async data not loaded | Is there a loading state? Does the API always return this field? |
| `NetworkError: 504 Gateway Timeout` | Backend timeout under load | Is it peak hours? Specific endpoint? |
| Console error on Safari only | Browser-specific API or CSS | Check MDN compatibility for the API in the stack trace |
| Error on page load, then works on refresh | Race condition in initialization | Check if it's the first session for new users |
| Error on mobile only | Viewport-dependent bug, touch event issue | Check responsive breakpoints at the error screen width |

## When It's a Regression

If the user suspects a deploy introduced the error:

1. Build a trend metric for the error event
2. Set the time range to span before and after the deploy date
3. Compute and look for a sharp inflection point at the deploy timestamp
4. If confirmed: "This error appeared immediately after the July 15th deploy — 0 occurrences before, 147 users affected since."

## Guidelines

- Never read session transcripts in the main context — use the `session-context` agent.
- When you find a session with an error, save the session URL. The user may need to send it to an engineer.
- If an error affects a single user, it's likely user-specific state. If it affects many, it's likely a code bug.
- Always check the device/browser breakdown — Safari-only or mobile-only errors point to specific code paths.
- If you can't find the root cause from 5 sessions, say so. Don't fabricate a theory from insufficient evidence.
