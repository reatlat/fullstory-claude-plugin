---
name: scroll-depth-analyzer
description: Analyze how far users scroll on key pages — content visibility, where users stop reading, and whether important content is being seen. Use when evaluating page layout, content effectiveness, or CTA placement.
---

# Scroll Depth Analyzer

Measure how far users scroll on your pages — where they stop, what they see, and whether your key content is actually being viewed.

## When to Use

- "Are users scrolling past the fold on the landing page?"
- "Do users reach the CTA at the bottom of the pricing page?"
- "Where do users stop reading on the blog post?"
- "Is the important content on /features actually being seen?"
- "Compare scroll depth between new and returning visitors"
- "Did the new page layout improve scroll depth?"

## Mental Model

Users don't read every word. They scan and scroll. The question is: do they scroll far enough to see the important stuff?

Scroll depth is measured in percentage of the page:
- **25%** — User saw the hero/header
- **50%** — User saw the first section of content
- **75%** — User saw most of the page
- **100%** — User reached the footer

The "fold" (visible area without scrolling) varies by device — it's roughly 600-800px on desktop, 500-600px on mobile. Content above the fold gets nearly 100% visibility. Content at 75% scroll depth might only be seen by 30% of users.

## Workflow

### Step 1: Choose the page

Which page does the user want to analyze? Typically:
- Landing pages (are users reading the pitch?)
- Pricing pages (do users see all tiers?)
- Blog posts (are users reading to the end?)
- Product pages (are users seeing the specs?)
- Checkout pages (are users seeing the full form?)

### Step 2: Build scroll depth metrics

Fullstory doesn't have a native scroll depth metric, so build it from page view and time-spent data:

```
fullstory:build_metric(
  query="users who viewed /pricing",
  output_type="single_number"
) → total visitors

fullstory:build_metric(
  query="users who spent >30 seconds on /pricing",
  output_type="single_number"
) → engaged visitors (read beyond the fold)

fullstory:build_metric(
  query="users who spent >60 seconds on /pricing",
  output_type="single_number"
) → deep readers (read most of the page)
```

Time on page is a proxy for scroll depth — longer time = more scrolling.

### Step 3: Check CTA visibility

For the CTA or key content on the page:
```
fullstory:build_metric(
  query="users who viewed /pricing AND clicked the 'Start Trial' CTA",
  output_type="single_number"
)
→ CTA click rate = clicks / total visitors
```

Low CTA click rate + high time on page = users read the page but the CTA didn't convince them. Low CTA click rate + low time on page = users never scrolled to the CTA.

### Step 4: Session-level scroll analysis

Pull sessions for the page and use `session_view` to check what users actually see:
```
fullstory:get_sessions(page="/pricing", limit=5)
→ session_view at timestamps: 5s (what's above the fold), 30s (what they scrolled to), 60s (how far they got)
```

Key things to check:
- Is the CTA visible at 30s? If not, users need to scroll to find it.
- Is important content cut off at the fold? Users may think the page ends.
- Are there "false bottoms" (large white spaces that make users think the page is done)?

### Step 5: Segment scroll behavior

Compare scroll depth across cohorts:
```
By device:
Desktop: 45% reach 75%+ scroll depth
Mobile: 28% reach 75%+ scroll depth → users scroll less on mobile (fatigue)

By source:
Organic search: 52% reach 75%+
Paid/landing page: 31% reach 75%+ → landing page visitors are less committed
```

### Step 6: Report

```
## Scroll Depth — /pricing (last 30 days, 8,400 visitors)

### Visibility Estimates
- Above fold (0-25%): ~100% of visitors see this content
- Mid-page (25-50%): ~65% of visitors (spent >15s)
- Lower page (50-75%): ~40% of visitors (spent >30s)
- Bottom (75-100%): ~25% of visitors (spent >60s)

### Critical Finding
The "Enterprise" pricing tier is at ~70% scroll depth — only ~35% of visitors see it. Your most expensive plan is invisible to 2/3 of potential buyers.

### CTA Performance
- "Start Free Trial" (above fold): 8.2% CTR
- "Contact Sales" (at 75% scroll, enterprise section): 1.1% CTR
→ Sales CTA is performing poorly, but it's at the bottom of the page. Move it higher.

### By Device
- Desktop: 45% reach enterprise section
- Mobile: 22% reach enterprise section (page is longer on mobile due to stacking)

### Recommendations
1. Move enterprise tier higher on the page (above 50% scroll depth)
2. Add a second "Contact Sales" CTA above the fold
3. On mobile, collapse the comparison table (it's pushing enterprise tier way down)
```

## Scroll Depth Heuristics

| Time on Page | Estimated Scroll Depth | User Intent |
|-------------|----------------------|-------------|
| < 10 seconds | 0-25% | Bounced — didn't engage |
| 10-30 seconds | 25-50% | Scanned — got the gist |
| 30-90 seconds | 50-75% | Read — engaged with content |
| 90+ seconds | 75-100% | Deep read — high intent |

## Guidelines

- Fullstory doesn't natively track scroll depth as a percentage. Time-on-page is a proxy, not a direct measurement. Be transparent: "I'm estimating scroll depth from time on page and session screenshots."
- The "fold" varies wildly by device. Desktop at 1440p sees 900px above the fold. iPhone SE sees 500px. Content placement needs to account for the lowest common denominator.
- A CTA at the bottom of a long page isn't failing because of the copy — it's failing because 75% of users never see it. Don't A/B test the button color. Move it up.
- False bottoms are a real problem — a large image or whitespace that looks like the end of the page. If users consistently stop at a specific point, check `session_view` at that scroll position to see what it looks like.
- For blog posts, scroll depth correlates with content quality. If readers consistently stop at 30%, the intro isn't compelling. If they read to 90%, the content is good — optimize for conversion at the end.
