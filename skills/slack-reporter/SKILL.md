---
name: slack-reporter
description: Format Fullstory findings for Slack — structured summaries with key stats, session links, and action items ready to paste into team channels. Use when the user wants to share findings with their team in Slack.
---

# Slack Reporter

Format Fullstory findings as clean, scannable Slack messages — key stats, session evidence, and action items. Ready to copy-paste into team channels.

## When to Use

- "Send this to #product-engineering on Slack"
- "Format the weekly digest for Slack"
- "Share the frustration analysis with the team"
- "Post an incident update to #incidents"
- "Summarize the experiment results for #growth"

## Mental Model

Slack messages are scannable, not readable. Nobody reads paragraphs in Slack. A good Slack report:
- Lead with the headline (what happened, in one sentence)
- Show the numbers (3-5 key stats, max)
- Link to evidence (session URLs, metric URLs)
- End with a clear ask or action item

## Workflow

### Step 1: Decide what to report

What just happened?
- Investigation completed (frustration analysis, error forensics, funnel analysis)
- Weekly digest ready
- Incident detected
- Experiment concluded
- Anomaly found

### Step 2: Extract the headline

One sentence that tells the story:
- "Rage clicks on 'Apply Promo Code' are up 23% week-over-week — 412 users affected, concentrated on mobile checkout."
- "Checkout conversion dropped 4 percentage points after the July 15 deploy. Root cause: API timeout on /api/orders during peak hours."
- "Weekly digest Aug 1-7: product is healthy overall, one new error on /dashboard worth investigating."

### Step 3: Format the message

Template:
```
📊 [Headline]

• 412 users affected (↑23% WoW)
• 73% mobile, concentrated on /checkout
• Avg 6 rage clicks per session before abandonment

🔗 [Session](https://app.fullstory.com/ui/org/session/abc123) | [Metric](https://app.fullstory.com/ui/metric/xyz)

🧵 Root cause: The promo code toast says "applied" but the cart total doesn't update. Users think the promo failed and retry repeatedly.

📋 Recommended: Increase "Apply" button touch target (currently 32px, below 44px minimum) and fix the toast-to-cart-total disconnect. @frontend-team can take this.

cc @product
```

### Step 4: Adapt for context

**Incident alert** (#incidents):
```
🚨 INCIDENT: /api/orders returning 504 for ~15% of requests
• Started: 19:00 UTC, ongoing
• Affected: ~200 users in last hour
• Impact: Checkout broken for affected users
• [Session evidence](https://...)
• Engineering investigating — updates in thread 🧵
```

**Weekly digest** (#product-weekly):
```
📊 Weekly Digest — Aug 1-7

✅ Overall healthy week
• Traffic: 1.2M page views (+3%)
• Conversion: 21% (stable)
• Errors: 0.8% error rate (stable)

🟡 1 thing to watch:
• New TypeError on /dashboard — 89 users affected
  → [Investigation thread](link)

Full dashboard: [Fullstory](link)
```

**Experiment result** (#growth):
```
🧪 Checkout Redesign — Final Results

Variant beats control: +19.8% conversion lift (3.6pp)
• Control: 18.2% (3,640/20,000)
• Variant: 21.8% (4,360/20,000)
• 14-day experiment, 40K users, result is directionally strong

⚠️ Mobile rage clicks up 12% in variant — review before shipping
🔗 [Full analysis](link)

Recommendation: Ship with mobile rage-click fix included.
```

### Step 5: Deliver

Present the formatted message as a code block the user can copy. Offer: "Want me to add anything, change the tone, or tag specific people?"

## Slack Formatting Rules

| Element | Rule |
|---------|------|
| Headline | Always first. One sentence. Emoji for scanability. |
| Numbers | Bullet points (•). 3-5 max. |
| Links | Session URL and metric URL — always. |
| Root cause | One sentence. If unknown, say "under investigation." |
| Action | What should happen next? Who owns it? |
| Tags | @team or @person if relevant. cc for FYI. |
| Thread | "Updates in thread 🧵" for ongoing issues. |
| Length | Under 200 words. If longer, split into thread. |

## Guidelines

- The message is the product. Don't make the user edit your Slack post. They're copying it, not workshopping it.
- Always include session URLs and metric URLs. Engineers want to see the evidence.
- Emojis are scanability tools, not decoration. 📊 = report, 🚨 = incident, 🧪 = experiment, 🔴/🟡/🟢 = severity.
- Don't @here or @channel unless it's genuinely urgent. Ask the user first.
- For ongoing incidents, post an initial alert, then updates in the thread. Don't wait for the full investigation to post.
- If the user asks for a different channel or audience (exec vs engineers), reformat. Execs want impact and recommendation. Engineers want evidence and next steps.