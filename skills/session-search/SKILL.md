---
name: session-search
description: Search across sessions by user properties, device, browser, custom events, or page — find the needle in the haystack. Use when you need to find specific sessions without a pre-built metric or segment.
---

# Session Search

Find sessions by searching across user properties, device, browser, custom events, and pages — without needing a metric or segment first.

## When to Use

- "Find sessions where users on Safari mobile visited /checkout"
- "Show me sessions from enterprise users who saw an error"
- "Find the session for user ID 84721 from yesterday"
- "Show me sessions where custom event 'trial_started' fired"
- "Find sessions from users in Germany who visited the pricing page"

## Mental Model

Session search is for targeted lookups when you know *who* or *what* you're looking for. It's different from metrics (which aggregate across many users) and opportunities (which rank by frustration). Session search is a point query: "find me these specific sessions."

## Workflow

### Step 1: Clarify the search criteria

What does the user know? Common search axes:

| Criterion | Tool |
|-----------|------|
| Specific user ID or email | `fullstory:get_user_profile` → sessions |
| Device type or browser | `fullstory:get_sessions` with device/browser filters |
| Page URL | `fullstory:get_sessions` with page filter |
| Custom event | `fullstory:get_sessions` with event filter |
| User property (plan, role, country) | Build segment → `fullstory:get_sessions(segment_id)` |
| Time range | Always specify — narrow is faster |

### Step 2: Execute the search

**By user ID or email:**
```
fullstory:get_user_profile(userIdentifier="user@example.com")
→ returns user's recent sessions with device_id and session_id
→ load each session through session-context agent
```

**By device/browser/page:**
```
fullstory:get_sessions(
  device="mobile",
  browser="safari",
  page="/checkout",
  time_range="last_7_days",
  limit=5
)
```

**By user property (plan, role, country):**
```
fullstory:build_segment("users on enterprise plan in Germany")
fullstory:get_sessions(segment_id=segment_id, limit=5)
```

**By custom event:**
```
fullstory:get_sessions(
  event="trial_started",
  time_range="last_30_days",
  limit=5
)
```

### Step 3: Present results

For each session found, show:
- Session date/time
- User identifier (if available)
- Device and browser
- Key pages visited
- Session URL (so the user can open it in Fullstory)

If the search returns many sessions, offer to narrow by:
- Adding a time range constraint
- Adding a device/page filter
- Combining criteria (e.g., "enterprise users on mobile who visited /checkout")

### Step 4: Investigate (if needed)

If the user wants to know what happened in a session, pass it to `session-review`. If they want to know what a specific user did across multiple sessions, use `user-journey`.

## Common Search Patterns

| Question | Approach |
|----------|----------|
| "Show me user 84721's sessions" | `get_user_profile` → sessions |
| "Find Safari mobile sessions on /checkout" | `get_sessions(device="mobile", browser="safari", page="/checkout")` |
| "Enterprise users who hit errors" | Build enterprise segment + build error metric → `get_sessions(metric_id)` |
| "Sessions where trial_started fired" | `get_sessions(event="trial_started")` |
| "Sessions from Germany on pricing page" | Build Germany segment → `get_sessions(segment_id, page="/pricing")` |

## Guidelines

- Narrow time ranges first — scanning 90 days of sessions is slow. Default to `last_7_days`.
- If search returns zero, broaden the criteria (wider time range, fewer filters) before reporting "not found."
- Always return session URLs — the user may want to watch the replay in Fullstory.
- For user property searches (plan, role, country), building a segment is one extra call but much faster than scanning all sessions.
- If the user wants to understand *what happened* in the sessions found, delegate to `session-review` or `session-context` — don't try to do it here.
