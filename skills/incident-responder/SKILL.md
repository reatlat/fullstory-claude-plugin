---
name: incident-responder
description: Structured incident response workflow — detect, investigate, communicate, and resolve using session data as evidence. Use when an incident is declared, a production issue is reported, or the on-call engineer needs session evidence.
---

# Incident Responder

Structured incident response using Fullstory session data — detect the blast radius, find affected users, provide evidence to engineering, and communicate status.

## When to Use

- "We have an incident — checkout is down"
- "Users are reporting payment failures — investigate"
- "Declare an incident for the API errors on /dashboard"
- "What's the blast radius of the 504 errors?"
- "Update the incident status for the team"

## Mental Model

An incident has a lifecycle:
1. **Detect**: Something is wrong (alert fired, user report, anomaly detected)
2. **Triage**: How bad is it? How many users? What's the blast radius?
3. **Investigate**: What's the root cause? Session evidence.
4. **Communicate**: Tell the team. Status updates.
5. **Resolve**: Fix deployed, verified.
6. **Retrospect**: What happened? How do we prevent it?

Fullstory MCP helps with steps 1-4 and 6. Step 5 (the actual fix) is on engineering.

## Workflow

### Step 1: Detect and declare

If the incident was detected by `anomaly-detector`, `predictive-alerts`, or a user report:

**Declare the incident** with severity:
- **SEV1** — Critical: Core functionality broken for all users (checkout down, login broken, data loss)
- **SEV2** — High: Core functionality broken for some users or secondary feature broken for all
- **SEV3** — Medium: Non-critical feature broken, workaround exists
- **SEV4** — Low: Cosmetic issue, edge case, investigation-only

### Step 2: Triage — measure blast radius

Quickly quantify impact:
```
fullstory:build_metric(
  query="checkout completion rate",
  output_type="single_number"
)
fullstory:compute_metric(metric_id, time_range="last_1_hour")
→ if 0% or near-zero, checkout is completely broken (SEV1)
```

Key triage questions:
- How many users are affected? (absolute count)
- What percentage of users? (of total traffic)
- Is it specific to device/browser/geo? (or everyone?)
- When did it start? (pinpoint the timestamp)
- Is it getting worse? (accelerating or stable?)

### Step 3: Investigate — find the root cause

```
fullstory:get_opportunities(time_range="last_1_hour")
→ any new frustration signals coinciding with the incident?

fullstory:build_metric(
  query="errors on /checkout in last 1 hour by type",
  output_type="top_n"
)
→ what specific error is occurring?

fullstory:get_sessions_for_opportunity(...)
→ pull sessions from affected users
→ session-context: "What happened when this user tried to checkout? What error did they see? What was the last successful action?"
```

Cross-reference with:
- `deploy-radar`: Did a recent deploy cause this?
- `api-monitor`: Is a backend endpoint failing?
- `error-forensics`: What's the specific error and stack trace?

### Step 4: Communicate — status updates

Use `slack-reporter` to format updates for the team:

**Initial alert** (within 5 minutes of detection):
```
🚨 SEV1: Checkout is down — 0% completion rate since 14:32 UTC
• All users affected (not device/browser specific)
• Started immediately after v2.3.1 deploy at 14:30
• Suspected cause: deploy regression on /api/orders
• Engineering investigating — updates in thread 🧵
• [Metric](link) | [Sessions](link)
```

**Update** (every 30 minutes):
```
🔄 SEV1 Update — 15:00 UTC (T+28min)
• Root cause confirmed: /api/orders returning 500 after deploy
• Fix identified: rollback deploy to v2.3.0
• Rollback in progress — ETA 15:10
• 1,200 users affected so far
```

**Resolution**:
```
✅ SEV1 RESOLVED — 15:12 UTC (T+40min)
• Deploy v2.3.1 rolled back to v2.3.0
• Checkout conversion returning to normal (18% and climbing)
• 1,400 total users affected during 40-minute outage
• Postmortem scheduled for tomorrow 10:00
• [Incident doc](link)
```

### Step 5: Verify resolution

After the fix is deployed:
```
fullstory:build_metric(query="checkout completion rate", output_type="single_number")
fullstory:compute_metric(metric_id, time_range="last_30_minutes")
→ confirm conversion is back to baseline
```

Check for:
- Conversion rate back within normal range
- Error rate dropped to baseline
- No new frustration signals introduced by the fix

### Step 6: Retrospect

After the incident is resolved, prepare a summary:
- What happened? (timeline)
- What was the root cause?
- How many users were affected? (blast radius)
- What was the business impact? (revenue estimate via `revenue-impact`)
- How was it detected? (alert, user report, anomaly)
- How long to detect, triage, resolve? (MTTD, MTTR)
- What prevented faster detection/resolution?
- Action items to prevent recurrence

## Incident Commands Quick Reference

| Command | What it does |
|---------|-------------|
| "Declare SEV1 — checkout is down" | Starts incident response workflow |
| "What's the blast radius?" | Quantifies affected users, devices, geo |
| "Find sessions from affected users" | Pulls session evidence for engineering |
| "Update the incident — we're rolling back" | Formats a Slack status update |
| "Verify the fix — is checkout working?" | Checks metrics post-fix |
| "Generate the postmortem" | Creates incident summary with timeline and metrics |

## Guidelines

- Speed over perfection in triage. A rough blast radius estimate in 2 minutes is better than a precise count in 20. Engineering needs to know: "everyone or some users?"
- Always check for a recent deploy. 60-70% of incidents are deploy-related. `deploy-radar` should be the first thing you check.
- Session evidence is gold for engineering. "Checkout is broken" is less useful than "Users see '504 Gateway Timeout' after clicking 'Place Order' — here's a session URL showing the error."
- Don't declare severity without asking. SEV1 means drop everything. If you're unsure, say "This looks like SEV1 or SEV2 — how do you want to classify it?"
- After resolution, always verify. A fix that "should work" is not the same as a fix that's confirmed working through session data.
- Postmortems should be blameless. Focus on "what in our process allowed this to happen?" not "who caused this?"