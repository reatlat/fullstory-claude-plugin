---
name: jira-bug-reporter
description: Create detailed bug reports from session evidence — auto-populate Jira tickets with session URLs, error messages, device info, and reproduction steps. Use when the user wants to file a bug from a Fullstory session.
---

# Jira Bug Reporter

Turn session evidence into detailed bug reports — auto-populated with session URLs, error messages, device info, stack traces, and reproduction steps.

## When to Use

- "File a bug for the checkout error we just found"
- "Create a Jira ticket from this rage-click session"
- "Write up the TypeError on /dashboard as a bug report"
- "Report the mobile Safari checkout issue to the engineering team"

## Mental Model

You have raw data from Fullstory — session URLs, error messages, timestamps, device info — and you need to turn it into a structured bug report an engineer can act on. The skill formats everything, then the user pastes it into Jira (or you output it as a markdown block ready to copy).

If the user has Jira MCP or a Jira API integration, you can create the ticket directly. Otherwise, you produce a formatted report they can copy-paste.

## Workflow

### Step 1: Gather evidence

Before writing the report, collect everything:

From `error-forensics` or `frustration-hunter`:
- Error message and stack trace
- Page URL where it occurred
- Device and browser (with versions if available)

From `session-review`:
- Session URL (most important — engineer can watch the replay)
- Timestamp of the error/frustration event
- Screenshots from `session_view` showing the UI at failure
- What the user did before the error (reproduction steps)
- What happened after (did they retry? abandon? work around it?)

From user context:
- How many users are affected? (from `get_opportunity_stats` or `compute_metric`)
- Is this a new regression or an existing issue?
- Severity: cosmetic, functional, or data-loss?

### Step 2: Classify severity

| Severity | Definition | Example |
|----------|-----------|---------|
| **P0 – Critical** | Data loss, security, complete outage | Checkout broken for all users |
| **P1 – High** | Core functionality broken for many users | Rage-click on primary CTA, 400+ users |
| **P2 – Medium** | Feature broken for some users | Error on settings page, Safari-only, 50 users |
| **P3 – Low** | Cosmetic issue, edge case | Dead click on rarely-used icon, 5 users |
| **P4 – Trivial** | Visual polish, nice-to-have | Alignment issue on mobile, no functional impact |

### Step 3: Write the report

Template:

```
## Bug: [One-line summary of the issue]

**Severity**: P1 – High
**Affected users**: 412 (last 7 days)
**First seen**: July 15, 2026 (coincides with v2.3.0 deploy)
**Device/Browser**: 73% mobile, concentrated on Safari iOS 17+

### Steps to Reproduce
1. Navigate to /checkout
2. Enter a valid promo code in the "Promo Code" field
3. Click "Apply"
4. Observe: toast says "Promo applied" but cart total does not change
5. User clicks "Apply" 4-8 more times, then abandons checkout

### Expected Behavior
Clicking "Apply" with a valid promo code should update the cart total to reflect the discount.

### Actual Behavior
Toast displays "Promo applied" confirmation, but the total remains unchanged. The promo is not actually applied.

### Evidence
- Session: https://app.fullstory.com/ui/org/session/abc123 (timestamp: 14:23:05)
- Error: No console error — the UI confirms success but the action fails silently
- Screenshot: [from session_view at 14:23:05 showing the misleading toast]

### Root Cause (hypothesis)
The toast is triggered by the UI optimistically, before the API confirms the promo was applied. The API returns success but the cart state isn't re-fetched.

### Additional Notes
- Not browser-specific (reproduced on Chrome and Safari)
- May be related to PR #4521 (checkout refactor, merged July 14)
- See [Fullstory metric](https://app.fullstory.com/ui/metric/xyz) for aggregate data
```

### Step 4: Deliver

If the user has Jira MCP connected: create the ticket directly with the formatted content.

If not: output the report as a formatted block ready to copy.

Offer: "Want me to also pull 2 more sessions to confirm the reproduction steps work consistently across users?"

## Guidelines

- The session URL is the single most important piece of evidence. Always include it.
- Don't guess at the root cause unless you have strong session evidence. Label hypothesis clearly.
- Include the number of affected users — it helps the engineering team prioritize.
- If the issue only affects specific devices/browsers, highlight that prominently. "Safari-only" bugs go to different people than "all browsers."
- If you know the relevant PR or deploy, mention it. "Coincides with v2.3.0 deploy (July 14)" helps engineers find the breaking change.
- Don't assign severity without asking the user unless it's obvious (P0 = checkout broken for everyone, P3 = cosmetic).
