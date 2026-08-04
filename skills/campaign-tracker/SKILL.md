---
name: campaign-tracker
description: Track marketing campaign performance — UTM-tagged traffic, landing page conversion, campaign cohort behavior, and ROI signals. Use when analyzing campaign performance, measuring landing page effectiveness, or comparing marketing channels.
---

# Campaign Tracker

Track marketing campaigns end-to-end — from UTM click to conversion, with cohort-level behavior analysis.

## When to Use

- "How did the email campaign perform?"
- "Compare conversion rates across Facebook, Google, and email campaigns"
- "What do users from the Q3 product launch campaign do after landing?"
- "Which campaign has the best retention — users who convert and come back?"
- "Show me sessions from users who clicked the July newsletter"
- "Is paid search traffic converting better than organic?"

## Mental Model

A campaign sends users to your site with tracking parameters (UTM tags). Fullstory captures the landing page URL including those parameters. By building segments from UTM values, you can track campaign cohorts through every page and action.

The campaign funnel:
1. **Click**: User clicks the campaign link (happens outside Fullstory)
2. **Landing**: User arrives on your site (first page view with UTM params)
3. **Engagement**: User browses, interacts, explores
4. **Conversion**: User completes the goal (purchase, signup, demo request)

Fullstory sees steps 2-4. Step 1 data comes from your marketing platform.

## Workflow

### Step 1: Identify campaign parameters

Ask the user for the UTM parameters they want to track:
- `utm_source` (e.g., facebook, google, newsletter)
- `utm_medium` (e.g., cpc, email, social)
- `utm_campaign` (e.g., q3_launch, black_friday, summer_sale)
- `utm_content` (e.g., banner_a, hero_cta — for A/B testing creative)

### Step 2: Build campaign segments

Build a segment for each campaign cohort:
```
fullstory:build_segment(
  "users who first landed with utm_campaign=q3_launch in last 30 days"
)
```

For multi-campaign comparison, build one segment per campaign:
```
seg_q3_launch = "utm_campaign=q3_launch"
seg_black_friday = "utm_campaign=black_friday"
seg_newsletter = "utm_source=newsletter"
```

### Step 3: Measure campaign performance

For each campaign, compute:
```
fullstory:build_metric(query="page views", output_type="single_number")
fullstory:update_metric(metric_id, segment_id)
fullstory:compute_metric(metric_id)
```

Key metrics per campaign:
- **Traffic volume**: How many users landed?
- **Bounce rate**: Users who landed and left immediately (1 page view, <10 seconds)
- **Engagement**: Average pages per session, time on site
- **Conversion rate**: Completed the goal action (purchase, signup, etc.)
- **Cost per conversion** (if the user provides spend data)

### Step 4: Compare channels

Present side by side:

| Campaign | Users | Bounce Rate | Conversion | Conv Rate | Notes |
|----------|-------|-------------|------------|-----------|-------|
| q3_launch (email) | 5,200 | 42% | 312 | 6.0% | Highest volume, decent conversion |
| black_friday (paid) | 3,100 | 38% | 279 | 9.0% | Best conversion rate |
| summer_sale (social) | 1,800 | 68% | 36 | 2.0% | High bounce — landing page mismatch? |
| newsletter_jul (email) | 4,400 | 45% | 220 | 5.0% | Solid, consistent |

### Step 5: Analyze campaign cohorts

Campaign users may behave differently than organic users. Check:
- **Feature usage**: Do campaign users explore different features?
- **Retention**: Do they come back after the campaign, or are they one-and-done?
- **Frustrations**: Do campaign landing pages have unique UX issues?

```
fullstory:build_segment("users who converted from black_friday campaign")
fullstory:get_sessions(segment_id, limit=5)
→ session-context: "Did this user encounter any frustrations on the landing page? What did they do after converting?"
```

### Step 6: Report

```
## Campaign Report — July 2024

### Overall
- 4 active campaigns, 14,500 total landing users
- 847 conversions (5.8% overall conversion rate)

### Top Performer: black_friday (paid)
- 9.0% conversion rate, $45 CPA (per Google Ads)
- Users from this campaign have 2.3x higher retention than organic
- Landing page /black-friday performs well — no UX issues detected

### Underperformer: summer_sale (social)
- 2.0% conversion, 68% bounce rate
- Root cause: landing page /summer-sale has a broken image carousel (rage clicks detected)
- Users bounce within 8 seconds on average
- Fix the carousel or redirect to a working landing page

### Recommendations
1. Fix /summer-sale carousel immediately (hurting social campaign)
2. Increase paid budget for black_friday — strong ROI signal
3. Email campaigns perform consistently — maintain cadence
```

## Campaign-Specific Checks

| Check | Why | Tool |
|-------|-----|------|
| Bounce rate >50% | Landing page mismatch or technical issue | Check session transcripts — what do users see? |
| Converter retention <20% | Campaign attracted wrong audience | `retention-analyzer` on the campaign segment |
| Mobile conversion << desktop | Landing page broken on mobile | `page-performance` with device breakdown |
| Rage clicks on landing page | Broken element, confusing CTA | `frustration-hunter` on the landing page URL |

## Guidelines

- UTM parameters are on the first page view only. If a user returns later via a direct visit, they won't have UTM tags — but they're still in the campaign segment (because `first_seen` with UTM params attaches permanently).
- Always show conversion as both a rate and an absolute number. "6% conversion" with 10 users is different from "6% conversion" with 10,000.
- Bounce rate is hard to measure precisely in Fullstory. Use "1 page view AND <10 seconds" as a proxy, but caveat it.
- Campaign spend data comes from the marketing platform (Google Ads, Facebook Ads, email tool), not Fullstory. Ask the user for CPA/spend if they want ROI calculations.
- If a campaign landing page has UX issues, flag it immediately — that campaign is burning money on clicks that land on a broken page.
