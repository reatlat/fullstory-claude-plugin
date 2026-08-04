---
name: customer-360
description: Full 360-degree view of a specific customer — sessions, purchases, errors, support interactions, engagement history. Use when support needs context on a customer, sales wants to understand a prospect's behavior, or you need to investigate an escalated issue.
---

# Customer 360

Complete view of a single customer — all their sessions, key actions, frustration events, purchases, and engagement history in one place.

## When to Use

- "Show me everything about customer user@example.com"
- "What's the history of user 84721? Support escalated their ticket."
- "Sales wants to know if Acme Corp has been evaluating our product — check their sessions."
- "This enterprise customer is threatening to churn — what's their experience been?"
- "Onboard a new CSM — give them the full picture of this account."

## Mental Model

A Customer 360 is a structured snapshot of one user across all dimensions available in Fullstory:

- **Identity**: Who they are (email, plan, role, account)
- **Engagement**: How often they use the product, what features
- **Success**: Did they achieve their goals (conversions, completions)?
- **Friction**: What frustrated them (errors, rage clicks, dead clicks)?
- **Trajectory**: Getting better or worse over time?

## Workflow

### Step 1: Look up the customer

Start with whatever identifier you have:
```
fullstory:get_user_profile(userIdentifier="user@example.com")
→ returns user_id, display_name, email, plan, role, first_seen, last_seen, total_sessions
```

If the lookup returns nothing, try alternative identifiers or ask the user for a more specific one.

### Step 2: Get session history

```
fullstory:get_sessions(user_id, time_range="last_90_days", limit=50)
```

If many sessions, sample strategically:
- First 5 sessions (onboarding experience)
- Last 5 sessions (current state)
- Any sessions with errors or high frustration signals

### Step 3: Identify key events

Use `session-context` agent to scan each sampled session for:
- **Conversions**: purchases, signups, upgrades
- **Friction**: errors, rage clicks, dead clicks, form abandonment
- **Feature usage**: which features did they use? power user or casual?
- **Support indicators**: visited help pages, searched docs, contacted support

### Step 4: Check for frustration patterns

```
fullstory:get_opportunities → filter to this specific user's sessions
→ have they been experiencing recurring frustrations?
```

Or build a user-specific metric:
```
fullstory:build_metric(
  query="rage clicks and dead clicks by user 84721",
  output_type="single_number"
)
```

### Step 5: Assess engagement trajectory

Compare early sessions vs recent sessions:
- **Growing**: More sessions, deeper feature usage, exploring advanced features
- **Stable**: Consistent usage, same patterns
- **Declining**: Fewer sessions, shallower engagement, visiting cancellation/pricing pages
- **Churned**: No sessions in 30+ days despite previous engagement

### Step 6: Assemble the 360

```
## Customer 360 — user@example.com (User ID: 84721)

### Profile
- Plan: Enterprise | Role: Admin | Account: Acme Corp (50 seats)
- Customer since: March 2024 (17 months)
- Total sessions: 342 | Last active: Aug 3, 2026

### Engagement
- Last 30 days: 12 sessions (stable — matches their 6-month average)
- Primary features: Dashboard, Reports, User Management
- Power user: uses advanced filters, exports data weekly

### Success Events
- Last purchase: Jul 28 (order #4821, $1,200)
- 3 purchases in last 90 days ($3,800 total)
- No failed purchase attempts

### Friction Events
- 2 errors in last 90 days (both on /reports — likely a known bug, see JIRA-4521)
- 1 rage click event on "Export CSV" button (Jul 15 — button was slow to respond)
- No form abandonment, no dead clicks

### Trajectory: Stable ✅
Consistent enterprise user. No churn risk. One known bug affected them (JIRA-4521, fix in progress).

### Support Context
- Support ticket #2841 (Jul 15): "Export button not working" — resolved, caused by the JIRA-4521 bug
- No open tickets

### Recommended Actions
- Follow up when JIRA-4521 is deployed — this user was affected
- Good candidate for case study (long-term enterprise, high engagement)
```

## Guidelines

- The 360 is a synthesis, not a data dump. Curate — highlight what matters, skip the boring sessions.
- For enterprise/B2B customers, include account-level context if available (total seats, other users on the account).
- If the user has 100+ sessions, don't read them all. Sample: first 5, last 5, and any with errors.
- The trajectory assessment (growing/stable/declining/churned) is the most actionable insight for CS and sales.
- Privacy: don't expose PII in the output unless the user explicitly asked for it. Use masked identifiers in the report.
- If the customer has zero sessions, say so and suggest checking: "They may be using a different email, or they're a prospect who hasn't used the product yet."