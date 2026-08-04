---
name: heatmap-generator
description: Generate click and interaction heatmaps from session data — visualize where users click, tap, and hover on any page. Use when asking "what do users click on most", "are users clicking non-clickable elements", or "where should we place the CTA".
---

# Heatmap Generator

Generate interaction heatmaps from real session data — click density, tap zones, rage click hotspots, and dead click dead zones. No visual rendering (Fullstory doesn't expose pixel coordinates natively) — instead, element-level aggregation showing which elements get the most and least interaction.

## When to Use

- "What do users click on most on the homepage?"
- "Show me a click map of the pricing page"
- "Are users clicking non-interactive elements?"
- "Where should we place the CTA on the landing page?"
- "Which elements are being ignored?"
- "Compare click patterns between new and returning users"

## Mental Model

A heatmap visualizes interaction density. Hot zones = elements users interact with a lot. Cold zones = elements users ignore (or can't find). Rage click zones = elements users click repeatedly — often non-interactive elements that *look* clickable.

Fullstory doesn't expose cursor coordinates through the MCP, so you can't render a pixel-accurate heatmap. Instead, you generate an **element-level interaction map** — showing which DOM elements get clicked, how often, and in what pattern.

## Workflow

### Step 1: Choose the page

Which page? Landing page, pricing page, product page, checkout.

Specify page: `/pricing`, `/landing`, `/products/*`

### Step 2: Aggregate clicks by element

Build a metric for click events on the page:
```
fullstory:build_metric(
  query="click events on /pricing by element",
  output_type="top_n"
)
fullstory:compute_metric(metric_id)
```

This returns the most-clicked elements, ranked by click count.

### Step 3: Identify interaction patterns

**Hot elements** (high click count):
- Are these the intended CTAs? Good — users are finding them.
- Are these non-interactive elements? Bad — users think they're clickable but they're not (dead click risk).

**Cold elements** (low/no clicks):
- Is the primary CTA here? Bad — users aren't finding it.
- Is this decorative or secondary content? Good — expected.

**Rage click zones** (repeated clicks on same element):
```
fullstory:get_opportunities → filter by page → look for rage click signals
→ "Rage clicks on 'Apply Promo' button: 412 users, concentrated on /checkout"
```

**Dead click zones** (clicks on non-interactive elements):
```
fullstory:build_metric(
  query="clicks on non-interactive elements on /pricing",
  output_type="top_n"
)
→ "Dead clicks on pricing tier cards (not linked): 89 users — they think the card is clickable"
```

### Step 4: Analyze click sequence

What do users click before and after key elements?
```
fullstory:get_sessions(page="/pricing", limit=10)
→ session-context: "What did the user click before clicking 'Start Trial'? What did they click after?"
```

This reveals navigation patterns: "Users click 'Compare Plans' → scroll → click 'Start Trial'" — the comparison table drives the decision.

### Step 5: Segment the heatmap

Compare clicks by user type:
```
By device:
Desktop: CTA clicks concentrated on right-side hero button
Mobile: CTA clicks concentrated on sticky bottom bar

By source:
Organic: Users click "Features" → "Pricing" → "Start Trial"
Paid: Users click "Start Trial" immediately (or bounce)
```

### Step 6: Report

```
## Interaction Heatmap — /pricing (last 30 days, 8,400 visitors)

### Top Clicked Elements
1. "Start Free Trial" (hero CTA): 2,100 clicks (25% of visitors) 🔥🔥🔥
2. "Compare Plans" toggle: 1,800 clicks (21%) 🔥🔥
3. "Monthly/Annual" toggle: 1,200 clicks (14%) 🔥
4. Pricing tier cards (non-interactive!): 890 dead clicks (10.6%) ⚠️
5. FAQ accordion items: 650 clicks (7.7%)
6. "Contact Sales" (bottom): 210 clicks (2.5%) ❄️

### Critical Finding
Pricing tier cards are getting 890 dead clicks. Users think the entire card is clickable, but only the "Start Trial" button inside it works. The cards look like buttons but aren't.

### CTA Performance
- Hero CTA (above fold): 25% CTR — strong
- Bottom CTA (75% scroll depth): 2.5% CTR — nearly invisible
→ Move the "Contact Sales" CTA higher. Only 25% of visitors scroll far enough to see it.

### Rage Click Zones
- "Apply Promo" field on /checkout: hot rage click zone (separate page, but worth noting — users expect this to work)

### Segment Differences
- New visitors: 28% click hero CTA (higher — evaluating)
- Returning visitors: 18% click hero CTA (lower — already converted or not interested)
- Mobile: CTA clicks shift to sticky bottom bar (design is working as intended)

### Recommendations
1. Make pricing tier cards fully clickable (or remove the card styling that implies clickability)
2. Add a second "Contact Sales" CTA above the fold
3. Investigate "Apply Promo" rage clicks on /checkout (run `frustration-hunter`)
```

## Guidelines

- Element-level, not pixel-level. Fullstory MCP doesn't expose cursor coordinates. Be upfront about this — you're showing interaction density by element, not a visual heatmap overlay.
- Dead clicks on non-interactive elements are a top-3 UX issue on most sites. Always check for them.
- Cold CTAs are a placement problem, not a copy problem. If a CTA at the bottom of the page has 2.5% CTR, moving it up will do more than changing the button text.
- Rage click hot zones on interactive elements = the element is broken. Rage click hot zones on non-interactive elements = the element *looks* interactive but isn't. Different fixes.
- Cross-reference with `scroll-depth-analyzer` — if an element has low clicks AND is at a scroll depth most users don't reach, the problem is placement, not the element.
