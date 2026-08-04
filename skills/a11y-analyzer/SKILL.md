---
name: a11y-analyzer
description: Accessibility analysis from session data — keyboard-only navigation patterns, focus order issues, screen reader compatibility, and accessibility regressions. Use when auditing a11y compliance, investigating accessibility bugs, or validating inclusive design.
---

# Accessibility Analyzer

Analyze accessibility from real user sessions — keyboard navigation patterns, focus traps, missing labels, and a11y regressions. Complement automated audits with evidence from actual users.

## When to Use

- "Are keyboard-only users able to navigate our checkout?"
- "Did the redesign break accessibility for screen reader users?"
- "Find sessions where users tabbed through the page — did they get stuck?"
- "Are our form labels working for assistive technology?"
- "Validate WCAG compliance from real session data"
- "Show me sessions where users struggled with focus order"

## Mental Model

Automated a11y tools (Axe, Lighthouse) catch ~30% of accessibility issues. Real user sessions catch the rest — focus traps, keyboard dead ends, unlabeled interactive elements, content that changes without announcement.

Fullstory captures all DOM interactions. By looking at how users interact with your site *without* a mouse, you can identify where the experience breaks.

## Workflow

### Step 1: Identify a11y-relevant sessions

Build segments for users likely relying on assistive technology:
```
fullstory:build_segment("users who navigated using only keyboard (tab key usage, no mouse clicks)")
fullstory:build_segment("users who used form fields extensively (screen reader users fill forms via keyboard)")
```

Or filter sessions by interaction patterns:
```
fullstory:get_sessions(
  event="keydown",      # keyboard interactions
  exclude_event="click", # no mouse clicks = likely keyboard-only
  limit=10
)
```

### Step 2: Audit keyboard navigation

For each keyboard-navigated session, use `session-context` agent:
"What was the user's tab order? Did they tab through elements in a logical sequence? Did they get trapped in any element (modal, form, menu)? Were there dead tab stops where nothing was focused?"

Common keyboard issues:
- **Focus trap**: User tabs into a modal/dropdown and can't tab out
- **Skip link missing**: User tabs through 50 nav items before reaching content
- **Focus not visible**: Element receives focus but no focus ring — user doesn't know where they are
- **Tab order illogical**: Focus jumps from header to footer, skipping main content

### Step 3: Check form accessibility

For form-heavy pages, check:
```
fullstory:build_metric(
  query="users who interacted with /signup form using keyboard only",
  output_type="single_number"
)
```

With `session_view`, check:
- Are all form fields reachable via Tab?
- Do labels associate with inputs (clicking label focuses the input)?
- Are error messages announced (do they have `aria-describedby` or `role="alert"`)?
- Are required fields indicated both visually AND programmatically (`aria-required`)?

### Step 4: Check for focus-order regressions

After a UI deploy, compare keyboard navigation before/after:
```
fullstory:build_metric(query="sessions with keyboard navigation on /checkout", output_type="trend")
fullstory:compute_metric(metric_id, time_range="last_14_days")
→ look for a change in navigation patterns at the deploy timestamp
```

If users start spending more time tabbing or rage-clicking after deploy, something broke in the focus order.

### Step 5: Simple checks from session screenshots

Use `session_view` to visually verify at key interaction points:
- Focus indicator visible on the active element?
- Color contrast sufficient for the active element?
- Text readable without zooming on mobile?
- Interactive elements have clear affordance (looks clickable)?

### Step 6: Report

```
## Accessibility Audit — /checkout (last 30 days)

### Keyboard Navigation
- 142 sessions with keyboard-only navigation (estimated)
- Primary issue: Focus trap in promo code modal — 12 sessions show users tabbing repeatedly inside the modal without escape
- Tab order: Logical (Header → Shipping → Payment → Submit) ✅

### Form Accessibility
- All 8 form fields reachable via Tab ✅
- 3 fields missing explicit <label> associations: Promo Code, Gift Card, CVV 🔴
- Error messages not associated with fields (no aria-describedby) — screen reader users won't hear them 🔴
- Required fields visually indicated (red asterisk) but no aria-required ✅?

### Regressions
- No a11y regressions detected since Jul 15 deploy ✅

### WCAG Impact
| Issue | WCAG Criterion | Severity | Affected Users |
|-------|---------------|----------|---------------|
| Focus trap in modal | 2.1.2 No Keyboard Trap | High | 12 sessions (8%) |
| Missing form labels | 1.3.1 Info and Relationships | High | All keyboard users |
| Error announcement | 4.1.3 Status Messages | Medium | All form users |

### Recommendations
1. Add Escape key handler to promo code modal (WCAG 2.1.2)
2. Add explicit <label for="..."> to Promo Code, Gift Card, CVV fields
3. Add aria-describedby linking error messages to their fields
```

## Common A11y Issues Detectable in Sessions

| Session Signal | Likely A11y Issue | Fix |
|---------------|------------------|-----|
| Repeated Tab on same few elements | Focus trap (modal, menu, carousel) | Add Escape handler, focus trap management |
| Long pause, then rage click | Focus landed on invisible/inactive element | Check visibility of focused elements, add skip links |
| Click on label doesn't focus input | Missing `for` attribute on `<label>` | Add explicit label-input association |
| User tabs past important content | Heading/skip-link not keyboard accessible | Add `tabindex="-1"` for programmatic focus |
| Form submission with keyboard, then immediate correction | Error not announced to screen reader | Add `role="alert"` or `aria-live` to error container |
| Dead click on icon-only button | Missing `aria-label` or `title` on icon | Add accessible name to all interactive elements |

## Guidelines

- Fullstory can't directly detect screen reader usage — infer from keyboard-only interaction patterns. Be transparent about this limitation.
- Automated tools catch code-level issues. Your job is to find interaction-level issues that only appear in real usage.
- A single focus trap affecting 12 sessions out of 142 is an 8% failure rate — that's high enough to fail WCAG 2.1.2 conformance.
- Don't claim WCAG compliance or non-compliance. You're providing evidence, not a certification.
- Always cross-reference with the `mobile-analyzer` — many mobile a11y issues overlap (touch targets, zoom, text size).
- After any UI deploy, run a quick keyboard audit on the changed pages. Focus-order regressions are the most common deploy-caused a11y bug.
