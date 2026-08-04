---
name: metric-auditor
description: Audit existing metrics and segments for staleness, duplicates, bad configuration, and naming issues. Use when the user wants to clean up their Fullstory account, find redundant objects, or understand what already exists.
user-invocable: false
---

# Metric Auditor

Audit your Fullstory account — find duplicate metrics, stale segments nobody uses, misconfigured objects, and naming inconsistencies. Keeps the shared workspace clean so the team trusts the numbers.

## When This Runs

Invoked automatically when `general-analysis` finds multiple candidates for a search and needs to determine which one is canonical. Also runs when the user explicitly asks:
- "Are there duplicate metrics in our account?"
- "Clean up our segments — which ones are stale?"
- "What metrics exist for checkout?"
- "Is anyone actually using this segment?"

## Mental Model

Fullstory accounts accumulate cruft over time — metrics built for a one-off question, segments created for a meeting three months ago, copies of copies with slightly different names. This skill finds and diagnoses that cruft.

## Workflow

### Step 1: Discover everything

Search broadly to get the lay of the land:

```
fullstory:get_metric(regex="")  → list all metrics
fullstory:get_segment(regex="") → list all segments
```

If the account is large, narrow by keyword: `fullstory:get_metric(regex="checkout")`.

### Step 2: Check popularity

For every object found, call `fullstory:get_view_count` (up to 10 IDs at a time):

- **High view count**: Actively used — canonical, trusted.
- **Low view count**: Maybe stale — built once, never referenced again.
- **Zero views**: Almost certainly stale — safe to flag for cleanup.

### Step 3: Identify issues

**Duplicates**: Two metrics with nearly identical names and the same output type. Example: "checkout conversion" and "checkout conversion rate" both returning single_number. Flag: pick the one with higher view count as canonical.

**Stale objects**: Zero views and created more than 30 days ago. Flag for deletion.

**Naming problems**: "Untitled metric", "Segment 1", "Copy of checkout funnel". Suggest renaming.

**Misconfigured**: A segment that filters by a property that no longer exists, or a trend metric built as single_number. These are harder to detect without computing — flag if the name doesn't match the output type.

**Orphans**: A metric that references a segment that no longer exists (the `segment_id` was deleted). These will error on compute — flag them.

### Step 4: Present the audit

```
## Metric Audit

### Duplicates (2 pairs)
1. "checkout conversion" (247 views) vs "checkout conversion rate" (12 views)
   → Keep the first, delete or rename the second
2. "signup page views" (89 views) vs "Signup Page views" (3 views)
   → Keep the first (consistent casing), delete the second

### Stale (5 objects, 0 views)
- "Q1 experiment metric" (created Jan 15, never viewed)
- "Test segment for meeting" (created Mar 3, never viewed)
- ... (3 more)

### Naming issues (3 objects)
- "Untitled metric" → rename to describe what it measures
- "Segment 1" → rename to describe the cohort
- "Copy of checkout funnel" → merge with original or rename

### Total: 47 metrics, 23 segments. 5 objects flagged.

Want me to rename the canonic, delete the stale ones, or dig into any specific object?
```

### Step 5: Clean up (if user says yes)

For duplicates: rename the canonical one (if needed), flag the duplicate for deletion.
For stale objects: confirm before deleting — the user might know something you don't.
For naming: suggest a new name and apply it with `fullstory:update_metric` or `fullstory:update_segment`.

## Guidelines

- Never delete anything without explicit confirmation. Flagging is safe; deleting is not.
- When ranking by view count, treat anything with 5x or more views than the next candidate as canonical.
- If the account has >50 metrics or segments, don't list them all in the response. Summarize with stats (total, stale count, duplicate count) and offer to drill into specific categories.
- An object with zero views might be important — it could be brand new, or used via the API (which doesn't increment view count). Flag with a warning, don't assume it's garbage.
- Focus on actionable findings. A clean audit with nothing to fix is a good audit — report that and move on.
