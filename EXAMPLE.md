# Example: Claude + Fullstory in Action

A real conversation showing how Claude uses multiple skills from this plugin to investigate a product issue end-to-end. From "something feels off" to a Jira-ready bug report, in one conversation.

---

## Session Transcript

**User:** Something's wrong with checkout. Customers are complaining. Can you figure out what's going on?

**Claude** *(invokes `frustration-hunter`)*:
> Let me check what's frustrating users on checkout. I'll look at the last 7 days.

*[calls `fullstory:get_opportunities`, `fullstory:get_opportunity_stats`]*

> **Top frustration**: Rage clicks on "Apply Promo Code" — 412 users in 7 days, 73% mobile, concentrated on /checkout. That's up 23% vs last week. The next biggest frustration is only 87 users — this promo code issue is the outlier.

> Want me to pull some sessions and see what users are actually experiencing?

**User:** Yes, show me what's happening.

**Claude** *(invokes `session-review` + `session-context` agent)*:
> I pulled 5 sessions. Here's the pattern:

> 1. User enters a valid promo code in the field
> 2. Clicks "Apply"
> 3. A toast appears: "Promo applied" ✅
> 4. The cart total stays the same
> 5. User clicks "Apply" again... and again... (avg 6 clicks)
> 6. Eventually abandons checkout

> The toast is misleading. It confirms success before the promo is actually applied. Users have no way to know the promo failed.

**User:** How many people does this affect? And when did it start?

**Claude** *(invokes `deploy-radar` + `error-forensics`)*:
> 412 users this week, up from 335 last week (+23%). 

> *[checks deploy timeline]* The rage clicks started spiking on July 15 — same day as the v2.3.0 checkout deploy. No errors in the console, so it's not a crash. It's a logic bug — the toast fires optimistically before the API confirms.

**User:** What's this costing us?

**Claude** *(invokes `revenue-impact`)*:
> Of the 412 users who rage-clicked the promo button, checkout conversion for those users is 12% vs 22% for users who don't hit this issue. At your average order value of $85, that's roughly $3,500 in lost revenue this week alone — about $182K annualized if not fixed.

**User:** Ok, file a bug for the engineering team. And check if this affects mobile more than desktop — our mobile conversion has been down.

**Claude** *(invokes `jira-bug-reporter` + `mobile-analyzer`)*:

> **Bug filed**: "Promo code toast confirms success before promo is applied — causes rage clicks and checkout abandonment." Link: JIRA-4528

> **Mobile analysis**: You're right to ask. 73% of affected users are on mobile. Mobile checkout conversion is 12% vs 22% on desktop. The "Apply Promo" button is 32px tall on mobile (below the 44px minimum touch target). Users are rage-clicking it partly because it's hard to tap accurately.

> So it's a compound issue: the toast bug + a touch target that's too small. Fixing both could close most of the mobile-desktop conversion gap.

**User:** When we fix this, how will we know it worked?

**Claude** *(invokes `changelog-detective` + `predictive-alerts`)*:
> After the fix deploys, I'll run a before/after comparison: rage clicks on "Apply Promo", checkout conversion rate, and mobile conversion specifically. If the fix works, we should see rage clicks drop to near zero and conversion recover to baseline.

> I've also set a watch: if checkout conversion drops below 18%, or rage clicks on promo exceed 50/week, you'll get an alert.

**User:** Great. Send the team a summary in #product-engineering.

**Claude** *(invokes `slack-reporter`)*:

```
📊 Checkout Promo Code Bug — Investigation Summary

• 412 users affected this week (↑23% WoW)
• Root cause: Toast confirms promo success before API applies it
• Impact: ~$3,500/week lost revenue, 73% mobile

🔗 [Session evidence](url) | [JIRA-4528](url)

📋 Fix: await API confirmation before showing toast + increase button touch target to 48px
📋 After deploy: rage clicks should drop to near-zero

cc @frontend-team @product
```

---

## Skills Used in This Conversation

| # | Skill | What it did |
|---|-------|------------|
| 1 | `frustration-hunter` | Discovered the promo code rage click spike, ranked by impact |
| 2 | `session-review` + `session-context` | Pulled and analyzed 5 sessions, identified the toast/cart disconnect |
| 3 | `deploy-radar` | Correlated the spike with the July 15 deploy |
| 4 | `error-forensics` | Confirmed no console errors — it's a logic bug, not a crash |
| 5 | `revenue-impact` | Estimated $3,500/week lost revenue, $182K annualized |
| 6 | `jira-bug-reporter` | Created JIRA-4528 with session URLs, repro steps, severity |
| 7 | `mobile-analyzer` | Identified the compound mobile issue (touch target + toast bug) |
| 8 | `changelog-detective` | Set up before/after verification for the fix deploy |
| 9 | `predictive-alerts` | Set watch thresholds for post-fix monitoring |
| 10 | `slack-reporter` | Formatted the summary for #product-engineering |

**10 skills, one conversation, no SQL written.**
