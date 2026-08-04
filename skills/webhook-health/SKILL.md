---
name: webhook-health
description: Monitor FullStory webhook delivery — success rates, errors, latency, and webhook-driven integration health. Use when troubleshooting webhook failures, monitoring integration health, or verifying webhook configuration.
---

# Webhook Health

Monitor FullStory webhook delivery — is data flowing to your integrations, are webhooks failing, and are your downstream systems healthy?

## When to Use

- "Are our FullStory webhooks delivering successfully?"
- "Why didn't sessions show up in our data warehouse?"
- "Check webhook delivery since the configuration change"
- "Which webhook endpoints are failing?"
- "Monitor webhook health for the Slack and Jira integrations"

## Mental Model

FullStory can send session data, events, and alerts to external systems via webhooks. If webhooks fail, data stops flowing — your data warehouse misses sessions, your Slack channel stops getting alerts, your Jira integration stops creating tickets.

Webhook monitoring has two sides:
1. **Delivery**: Did FullStory successfully deliver the webhook? (HTTP 2xx response)
2. **Processing**: Did your downstream system correctly process the payload? (business logic failures)

FullStory can tell you about delivery. Processing failures need to be checked in the downstream system.

## Workflow

### Step 1: Check webhook configuration

Verify what's configured. Webhooks are typically set up in Fullstory's integration settings or API. Ask the user:
- "What webhook endpoints do you have configured?"
- "What events trigger them? (new session, new event, segment export, etc.)"
- "When were they last known to be working?"

If the user isn't sure, direct them to: Fullstory → Settings → Integrations → Webhooks

### Step 2: Check for delivery failures

Build a metric for webhook-related errors or check session data for failed webhook deliveries:
```
fullstory:build_metric(
  query="custom events related to webhook failures",
  output_type="single_number"
)
```

Or check if the data pipeline has gaps:
```
fullstory:build_metric(query="sessions created", output_type="trend")
fullstory:compute_metric(metric_id)
→ compare to your data warehouse: do the session counts match?
```

### Step 3: Verify integration health

For each downstream integration (Slack, Jira, data warehouse, Snowflake), verify data flow:

**Data warehouse (Snowflake, BigQuery, Redshift):**
- Compare session count in Fullstory vs warehouse: do they match for the same time period?
- Check for gaps: are specific hours or days missing?
- Check for duplicate sessions (webhook retry can cause duplicates)

**Slack/Teams notifications:**
- Did alerts fire for recent frustration signals? (Check `get_opportunities` → were any high-severity? → did Slack get notified?)
- Test: create a test annotation → did it trigger the webhook?

**Jira/Linear ticket creation:**
- When `jira-bug-reporter` creates a ticket, does it arrive?
- Check recent error sessions → were tickets created for P0/P1 issues?

### Step 4: Diagnose failures

If webhooks are failing, common causes:
- **Endpoint changed**: URL updated, webhook still pointing to old URL
- **Auth expired**: API key or token rotated, webhook auth not updated
- **Rate limiting**: Downstream system rate-limiting the webhook receiver
- **Payload changed**: Fullstory updated the webhook payload format, downstream parser broke
- **Network**: Firewall or VPC change blocking outbound webhook traffic
- **Timeout**: Downstream system slow to respond (>10s), Fullstory times out

### Step 5: Report

```
## Webhook Health — Last 7 Days

### Configured Webhooks
- Session export → Snowflake: 🟢 12,400 sessions delivered (0 failures)
- Frustration alerts → Slack: 🟡 8 sent, 2 failed (400 Bad Request — Slack channel archived?)
- Segment export → S3: 🟢 4 exports delivered (0 failures)
- Event stream → Kafka: 🔴 0 delivered since Aug 1 (endpoint unreachable)

### Data Pipeline Verification
- Fullstory sessions (last 7 days): 89,200
- Snowflake sessions (last 7 days): 89,198
- Gap: 2 sessions missing — within normal variance ✅

### Action Required
1. 🔴 Kafka endpoint unreachable since Aug 1 — check firewall/VPC rules
2. 🟡 Slack webhook failing for #product-alerts — channel may have been archived. Test the webhook URL.
```

## Webhook Troubleshooting Checklist

| Symptom | Check |
|---------|-------|
| Zero deliveries | Endpoint URL correct? DNS resolves? Firewall allows outbound? |
| Intermittent failures | Rate limiting? Timeout? Check downstream system logs. |
| Payload parsing errors | Did Fullstory change the webhook schema? Check changelog. |
| Duplicates | Retry logic working as expected? Check idempotency key handling. |
| Data gap (some hours missing) | Was there a Fullstory outage? Check status.fullstory.com. |

## Guidelines

- Fullstory MCP doesn't have a native `get_webhook_status` tool (as of the current beta). You're inferring webhook health from session data, metrics, and integration verification. Be transparent about this.
- A data gap between Fullstory and the warehouse doesn't always mean webhook failure — the warehouse might have ingestion lag. Check with the data team before raising alarms.
- Webhook authentication (API keys, tokens) is managed in Fullstory settings, not through the MCP. Direct the user there for auth configuration changes.
- If all webhooks are failing, don't investigate individual endpoints — check the common infrastructure: network, DNS, auth provider.
- For mission-critical webhooks (data warehouse, billing), recommend setting up external monitoring (Datadog, PagerDuty) rather than relying on manual checks.