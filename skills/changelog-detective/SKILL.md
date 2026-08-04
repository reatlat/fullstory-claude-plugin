---
name: changelog-detective
description: Detect what changed in the product by comparing before and after a deploy or date — find new behaviors, new errors, changed user flows, and unexpected side effects that weren't in the release notes. Use after a deploy or when the user suspects something changed but doesn't know what.
---

# Changelog Detective

Find what actually changed after a deploy — not what the release notes say, but what the session data shows. Catch undocumented changes, side effects, and surprises.

## When to Use

- "What actually changed after the deploy?"
- "Something feels different since Tuesday — what changed?"
- "Check for undocumented changes in the last release"
- "Did the deploy introduce any new user behaviors?"
- "What new errors appeared this week that weren't here last week?"

## Mental Model

Release notes tell you what was *intended* to change. Session data tells you what *actually* changed. These often differ. A "minor CSS fix" might accidentally break form validation. A "performance improvement" might introduce a new race condition. This skill hunts for the gaps between intent and reality.

## Workflow

### Step 1: Define the comparison window

Two time periods:
- **Before**: 24-72 hours before the deploy/change date
- **After**: 24-72 hours after (but not including the deploy window itself — things are chaotic during rollout)

Ask: "I'll compare 48 hours before and after the deploy (excluding the deploy hour). That OK?"

### Step 2: Before/After diff — Errors

What errors existed before vs after?
```
fullstory:build_metric(query="errors by type", output_type="top_n")
fullstory:compute_metric(metric_id, time_range=before)
fullstory:compute_metric(metric_id, time_range=after)
```

Look for:
- **New errors**: Error types that appear in After but not Before → these are deploy regressions
- **Resolved errors**: Error types in Before but not After → these are fixed (celebrate these)
- **Changed errors**: Same error type, different frequency → possible impact change

### Step 3: Before/After diff — Frustrations

```
fullstory:get_opportunities(time_range=before)
fullstory:get_opportunities(time_range=after)
→ compare the lists
```

Look for:
- **New frustration signals**: Rage clicks or dead clicks on elements that weren't problematic before
- **Resolved frustrations**: Frustrations that disappeared (fix worked!)
- **Changed patterns**: Same frustration, different page or device distribution

### Step 4: Before/After diff — User behavior

Did user flows change?
```
fullstory:build_metric(query="page views by page", output_type="top_n")
→ compare before/after: any pages suddenly getting more/less traffic?

fullstory:build_metric(query="conversion rate", output_type="single_number")
→ compare before/after: did conversion change?
```

Look for:
- **Traffic shifts**: Pages with significantly different traffic share
- **Flow changes**: New common page sequences (users navigating differently)
- **Time changes**: Sessions getting longer or shorter
- **Conversion changes**: Key funnels performing differently

### Step 5: Before/After diff — Device/Browser

Did the change affect specific platforms?
```
fullstory:build_metric(query="sessions by device type", output_type="top_n")
→ compare before/after: did the device mix change?

fullstory:build_metric(query="errors on Safari", output_type="single_number")
→ compare before/after: any browser-specific regressions?
```

### Step 6: Assemble the changelog

```
## Actual Changelog — v2.3.1 Deploy (Aug 4 15:00 UTC)

### What the release notes said:
- "Fixed promo code validation"
- "Updated checkout button styling"  
- "Performance improvements"

### What actually changed (from session data):

✅ **Promo code fixed**: Rage clicks on "Apply Promo" dropped from 412/week to 24/week (94% reduction)
✅ **Button styling**: No new dead clicks on checkout elements — visual change was clean
🟡 **Performance**: Page load time unchanged (was 2.1s, now 2.1s). "Improvements" may not be user-visible.

### Undocumented changes:

🔴 **New error introduced**: `TypeError: Cannot read property 'validate' of undefined` on /checkout — 34 users in 48 hours. Did not exist before deploy. Stack trace points to the new promo validation code.
🔴 **Mobile regression**: Checkout form fields shifted 20px down on iOS Safari. Not visible on desktop. Affects ~15% of mobile checkout users.
🟡 **New user behavior**: 8% of users now click "Apply Promo" before entering a code (dead click). The new button styling may make it look like it does something before the field is filled.

### Net assessment:
The deploy fixed the #1 frustration (promo code rage clicks) but introduced a new error and a mobile layout regression. Worth fixing both, but the fix was net positive — 412 fewer frustrated users per week vs 34 users seeing a new error.
```

## Changelog Diff Checklist

| Dimension | Before | After | Change |
|-----------|--------|-------|--------|
| Top 5 errors | [list] | [list] | New? Resolved? Changed frequency? |
| Top frustrations | [list] | [list] | New signals? Resolved? Shifted? |
| Conversion rate | X% | Y% | Δ = Y-X |
| Bounce rate | X% | Y% | Δ |
| Session duration | X min | Y min | Δ |
| Device mix | D/M/T% | D/M/T% | Shift? |
| Browser errors | Per browser | Per browser | New browser-specific issues? |
| Page traffic share | Top pages | Top pages | Any pages gaining/losing >20%? |

## Guidelines

- The deploy window (first hour after deploy) is noisy — caches warming, users getting new bundles, partial rollouts. Exclude it from before/after comparison. Start "after" at +1 hour.
- New errors are the strongest signal. An error that didn't exist before the deploy is almost certainly deploy-related. Prioritize these.
- Behavioral changes can be positive. If users are suddenly using a feature more, that might be intentional (better UX) rather than a bug. Investigate before flagging.
- Some changes are coincidental, not causal. A traffic spike might be a marketing campaign, not the deploy. Cross-reference with `campaign-tracker` before attributing to the deploy.
- If you find undocumented changes that look like bugs, flag them immediately. The team might not know. "Heads up: the deploy introduced a TypeError on /checkout that wasn't in the release notes. 34 users affected in 48 hours."
- For major deploys (50+ files changed), run this skill. For minor deploys (hotfix, config change), `deploy-radar` is usually sufficient.
