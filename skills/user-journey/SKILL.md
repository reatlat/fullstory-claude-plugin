---
name: user-journey
description: Trace a specific user's journey across sessions — what they did, where they struggled, and what happened before and after a reported issue. Use when investigating a specific user, reproducing a bug report, or understanding a high-value customer's experience.
---

# User Journey

Trace a specific user's path across multiple sessions — what they did, where they got stuck, and what happened before and after they reported an issue.

## When to Use

- "Show me everything user 84721 did this week"
- "A customer says the checkout broke yesterday — trace their sessions"
- "What did this user do before they churned?"
- "Show me the journey of our top 10 enterprise users"
- "Trace user@example.com's sessions for the last 30 days"

## Mental Model

A user journey spans multiple sessions — it's the story of one user across time, not a single session replay. You're looking for patterns: repeated frustrations, feature discovery, upgrade paths, churn signals.

## Workflow

### Step 1: Find the user

**By user ID or email:**
```
fullstory:get_user_profile(userIdentifier="user@example.com")
→ returns user_id, display_name, properties, and recent session metadata
```

Note the `user_id` — you'll use it for session searches.

If the user identifier isn't found, try:
- Search by partial match if the tool supports it
- Ask the user for a more specific identifier (full email, exact user ID)
- Try an alternative identifier (email if they gave you an ID, or vice versa)

### Step 2: Get session list

```
fullstory:get_sessions(user_id=user_id, time_range="last_30_days", limit=20)
```

If the user has many sessions, narrow by time range or look for sessions with notable events (errors, rage clicks).

### Step 3: Read session transcripts

For each session, use the `session-context` agent with a task that varies by what you're looking for:

**Bug investigation:**
"The user reported checkout broke yesterday. Did they visit /checkout? What error occurred? What was the last thing they did before the error?"

**Churn investigation:**
"What frustrations did this user encounter? Did they rage-click, hit errors, or abandon forms? What was their last session before they stopped using the product?"

**Power user analysis:**
"What features does this user engage with most? What's their typical flow? Are there any friction points in their advanced workflows?"

### Step 4: Synthesize the journey

Build a timeline:

```
User Journey — user@example.com (last 30 days)

Jul 28 — 4 sessions
  • Browsed /products, added 2 items to cart
  • Completed purchase (order #4821) — smooth, no issues

Jul 30 — 2 sessions
  • Searched for "return policy", visited /returns
  • Started return flow for order #4821, abandoned at step 3 (receipt upload)
  • Rage clicked "Upload Receipt" button 6 times — button was unresponsive

Aug 2 — 1 session  
  • Visited /returns again, completed return
  • No frustrations — likely figured out the receipt upload from a different device

Overall: 7 sessions, 1 frustration event (receipt upload button), 1 purchase, 1 return.
```

### Step 5: Report findings

Highlight:
- **Frequency**: How often do they use the product?
- **Key actions**: Purchases, signups, feature usage
- **Friction points**: Errors, rage clicks, dead clicks, abandonment
- **Trend**: Getting more or less engaged over time?
- **Notable gaps**: Long periods without sessions (churn risk)

## Multi-User Journeys

If the user asks about multiple users ("top enterprise accounts", "users who churned last month"):

1. Build a segment for the cohort
2. Call `fullstory:get_sessions(segment_id, limit=10)` to get sessions from multiple users
3. Group sessions by user ID
4. Synthesize per-user, then look for patterns across users

"A cross 5 churned enterprise users, the common pattern: all visited /billing in their last week, all hit the 'Update payment method' form, none completed it. The payment form appears to be broken for enterprise accounts."

## Guidelines

- Session transcripts are large — always use the `session-context` agent, never read them in the main context.
- For multi-session journeys, focus on the sessions with notable events (errors, rage clicks, conversions). Don't summarize boring sessions — flag them as "no notable events."
- If a user has >20 sessions in the time range, sample: the first 5, the last 5, and any with errors.
- When presenting the journey, include session URLs for the most important sessions so the user can watch the replay.
- Privacy: don't expose PII in your response unless the user explicitly asked for it. Use "user@..." or "user 84721" instead of full emails in the output.
