---
name: experiment-analyzer
description: A/B test and experiment analysis — measure statistical impact, segment breakdowns, and before/after comparisons. Use when the user is running an experiment, wants to know if a variant won, or needs to measure the impact of a change.
---

# Experiment Analyzer

Analyze A/B tests and experiments — measure impact, check statistical validity, break down results by segment, and tell you which variant won (and by how much).

## When to Use

- "Did the new checkout design improve conversion?"
- "Analyze the 'blue button vs green button' experiment"
- "Is the pricing page experiment statistically significant?"
- "Break down the experiment results by device type"
- "Which variant performed better for enterprise users?"
- "Should we ship the new onboarding flow?"

## Mental Model

An experiment compares two or more variants against a control. The goal is to determine:

1. **Direction**: Which variant performed better?
2. **Magnitude**: By how much? (+5%? +20%?)
3. **Confidence**: Is the difference real or noise? (statistical significance)
4. **Segments**: Does it work for everyone, or only certain users?
5. **Side effects**: Did the variant improve conversion but increase errors?

## Workflow

### Step 1: Define the experiment

Ask the user:
- **What's being tested?** (e.g., checkout flow redesign)
- **What's the success metric?** (e.g., checkout completion rate)
- **When did the experiment start?** (so you can scope the time range)
- **How are users split?** (50/50 random? By user property? By feature flag?)

If users are split by a user property (e.g., `experiment_group = "control"` or `"variant"`), use segments. If split by a feature flag that maps to a custom event, filter by that event.

### Step 2: Build metrics per variant

**For user-property splits** (experiment_group property):
```
fullstory:build_segment("users in experiment group 'control'") → seg_control
fullstory:build_segment("users in experiment group 'variant'") → seg_variant
fullstory:build_metric(query="checkout completion rate", output_type="single_number")
fullstory:update_metric(metric_id, segment_id=seg_control)
fullstory:compute_metric(metric_id) → control_result
fullstory:update_metric(metric_id, segment_id=seg_variant)
fullstory:compute_metric(metric_id) → variant_result
```

**For random splits** (no user property — use the experiment time window):
Build one metric scoped to the experiment's time range. Compute separately for control and variant periods if the experiment used time-based allocation (less common).

### Step 3: Calculate impact

```
Control:   18.2% conversion (3,640 / 20,000)
Variant:   21.8% conversion (4,360 / 20,000)

Absolute lift: +3.6 percentage points
Relative lift: +19.8%
```

Always show absolute AND relative. "3.6pp" tells you the real-world impact. "19.8%" tells you the proportional improvement.

### Step 4: Assess significance (with caveats)

Fullstory doesn't provide statistical significance natively. You can offer a heuristic:

- **Large sample, large effect** (>1,000 users per variant, >10% relative change): "The result is likely real — large sample and large effect."
- **Large sample, small effect** (>1,000 users, <5% relative change): "The improvement is small. It might be noise — consider running longer."
- **Small sample**: "Not enough data yet. You need at least 500–1,000 users per variant for a reliable read."

Always caveat: "Fullstory doesn't compute p-values. For rigorous statistical analysis, use an experimentation platform (Optimizely, LaunchDarkly, Statsig). I'm giving you directional guidance."

### Step 5: Segment the results

Check if the experiment affects different users differently:

```
fullstory:build_metric(
  query="checkout completion by device type",
  output_type="top_n"
)
→ compute for control segment, then variant segment
→ compare device breakdown between variants
```

Flag any segment where the variant performs *worse* than control — this is a regression risk.

### Step 6: Check side effects

Don't just measure the success metric. Check:
- Did errors increase in the variant?
- Did rage clicks/dead clicks change?
- Did page load time or form abandonment change?

Call `fullstory:get_opportunities` for both control and variant (using segments). Report: "The variant improved conversion by 19.8%, but rage clicks on the new form increased by 12%. Worth investigating before shipping."

### Step 7: Recommend

Based on the evidence:
- **Ship it**: Large, consistent improvement across all segments, no side effects.
- **Ship with caution**: Improvement in primary metric, but regressions in some segments. Flag which.
- **Iterate**: Improvement exists but is small. Suggest running another experiment.
- **Don't ship**: No improvement, or variant is worse.

## Guidelines

- Never claim statistical significance without a p-value. Use directional language: "The evidence suggests" not "The experiment proves."
- Always check for segment-level regressions — a variant that improves overall conversion but hurts mobile users should not ship.
- Side effects matter as much as the primary metric. An experiment that boosts conversion by 10% but doubles error rate is a net negative.
- If the experiment has only been running for <24 hours, warn that early results are unreliable (novelty effect, day-of-week bias).
- For experiments with revenue impact, invoke `revenue-impact` to attach dollar figures.
