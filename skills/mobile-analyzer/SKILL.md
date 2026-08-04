---
name: mobile-analyzer
description: Mobile-specific UX analysis — touch interactions, viewport issues, scroll behavior, mobile vs desktop patterns, and responsive design validation. Use when investigating mobile experience, responsive design bugs, or mobile conversion rates.
---

# Mobile Analyzer

Deep-dive into mobile user experience — touch events, viewport issues, responsive design bugs, and mobile-specific conversion patterns.

## When to Use

- "How does our mobile experience compare to desktop?"
- "Are mobile users having trouble with the checkout flow?"
- "Find sessions where mobile users encountered viewport issues"
- "Is the new responsive design working on tablets?"
- "What percentage of rage clicks happen on mobile vs desktop?"
- "Show me mobile sessions where users pinch-zoomed repeatedly"

## Mental Model

Mobile users are different from desktop users:
- **Input**: Touch targets, swipe gestures, virtual keyboard
- **Viewport**: Smaller screen, variable sizes, landscape/portrait
- **Context**: On-the-go, distracted, slower connections
- **Expectations**: Faster, simpler, thumb-friendly

A design that works on desktop can fail catastrophically on mobile — elements overlapping, touch targets too small, keyboard covering form fields, horizontal scroll appearing.

## Workflow

### Step 1: Quantify mobile vs desktop

Start with the split:
```
fullstory:build_metric(
  query="sessions by device type",
  output_type="top_n"
)
→ "68% desktop, 28% mobile, 4% tablet"
```

Then compare key metrics across devices:
```
fullstory:build_metric(query="checkout conversion by device type", output_type="top_n")
fullstory:compute_metric(metric_id)
```

### Step 2: Mobile-specific frustration signals

```
fullstory:get_opportunities → filter by device_type="mobile"
```

Mobile-specific signals to watch for:
- **Pinch-zoom events**: Users zooming in to read or tap — text too small, touch targets too small
- **Horizontal scroll**: Content wider than viewport — broken responsive layout
- **Orientation change rage**: Users rotating device repeatedly — layout broken in one orientation
- **Keyboard overlap**: Form fields obscured by virtual keyboard — user taps field, keyboard covers the submit button

### Step 3: Compare mobile vs desktop by page

For key pages, compare performance:
```
Page: /checkout
Mobile: 12% conversion, 34% bounce, 3 rage click events per 100 sessions
Desktop: 22% conversion, 18% bounce, 1 rage click event per 100 sessions
→ Mobile conversion is nearly half of desktop. The gap needs investigation.
```

### Step 4: Investigate mobile-specific sessions

```
fullstory:get_sessions(
  device="mobile",
  page="/checkout",
  metric_id=low_conversion_metric,
  limit=5
)
→ session-context agent: "What difficulties did this mobile user face on /checkout? Were form fields hard to tap? Did the keyboard cover content? Was any text cut off?"
```

### Step 5: Touch target analysis

Identify elements that users struggle to tap:
```
fullstory:get_opportunities → dead clicks on mobile → what elements?
```

Common touch target issues:
- Buttons < 44px (Apple HIG minimum)
- Links too close together (accidental taps)
- Hamburger menus that don't respond to tap
- Swipeable carousels that intercept vertical scroll

### Step 6: Report

```
## Mobile Experience Audit

### Overall Split
- Desktop: 68% | Mobile: 28% | Tablet: 4%
- Mobile share growing: +3% vs last quarter

### Mobile vs Desktop Performance
| Metric | Mobile | Desktop | Gap |
|--------|--------|---------|-----|
| Checkout conversion | 12% | 22% | -10pp 🔴 |
| Bounce rate | 34% | 18% | +16pp 🔴 |
| Rage clicks/100 sessions | 3.2 | 1.1 | +2.1x 🔴 |
| Avg session duration | 3m 12s | 6m 45s | -3m 33s |

### Critical Issues
1. **Checkout form on mobile**: The "Apply Promo" button is 32px tall (below 44px minimum). Users rage-click it 4-8 times. 73% of mobile checkout rage clicks target this button.
2. **Horizontal scroll on /products**: Content overflows viewport on iPhone SE (375px). 12% of mobile sessions on this page show horizontal scroll gestures.
3. **Tablet experience**: Only 4% of traffic, but 28% conversion — tablets convert better than desktop. Consider tablet-specific optimizations.

### Recommendations
1. Increase "Apply Promo" button to 48px minimum touch target
2. Fix horizontal overflow on /products for small viewports
3. Prioritize checkout mobile UX — closing the mobile-desktop conversion gap could add ~$12K/month in recovered revenue
```

## Mobile-Specific Debugging

| Issue | How to Detect | Fix |
|-------|-------------|-----|
| Touch target too small | Dead clicks on element (mobile only) | Min 44px touch target |
| Content cut off | Horizontal scroll events on mobile | Check viewport overflow at common breakpoints |
| Keyboard overlap | Form field focused → immediate page leave | Position key form fields above the keyboard zone |
| Unreadable text | Pinch-zoom events on text elements | Min 16px font size, proper viewport meta tag |
| Accidental taps | Two different links clicked simultaneously | Increase spacing between tappable elements |
| Slow mobile page load | High bounce rate + short time on page | Check mobile page speed (not in Fullstory — use PageSpeed Insights) |

## Guidelines

- Always compare mobile to desktop — a rage click rate of 3/100 sessions might seem normal until you see it's 1/100 on desktop.
- Mobile issues are often viewport-specific. Check multiple sizes: iPhone SE (375px), iPhone 14 (390px), Pixel (412px), iPad (768px).
- The `session_view` tool shows the actual rendered page at a specific viewport size — use it to verify layout bugs.
- If mobile conversion is significantly lower than desktop (2x gap or more), prioritize it. Mobile revenue is growing; a 2x conversion gap is a 2x revenue leak.
- Responsive design bugs after a deploy are one of the most common regressions. Run `deploy-radar` with mobile segment filtering after major UI deploys.
