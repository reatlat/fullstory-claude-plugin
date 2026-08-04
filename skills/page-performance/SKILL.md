---
name: page-performance
description: Page-level performance analysis — load times, errors by page, device breakdown, and page-to-page navigation patterns. Use when asking about specific page health, comparing pages, or finding pages with performance issues.
---

# Page Performance

Analyze page-level health — load times, errors, device breakdown, and how users navigate between pages.

## When to Use

- "Which pages have the most errors?"
- "How does /checkout perform on mobile vs desktop?"
- "What pages do users visit before /pricing?"
- "Show me sessions where /dashboard was slow to load"
- "Which pages have the highest abandonment rate?"
- "Compare all product pages by error rate"

## Workflow

### Step 1: Define the scope

What does the user want to know about the page(s)?
- **Error rate**: console errors, network failures on this page
- **Performance**: rage clicks, dead clicks, form abandonment on this page
- **Navigation**: what pages lead to/from this page
- **Device breakdown**: mobile vs desktop usage and error rates
- **Comparison**: rank multiple pages by a metric

### Step 2: Build page-scoped metrics

For a single page, scope the metric to that page:
```
fullstory:build_metric(
  query="console errors on /checkout",
  output_type="trend"   # or single_number or top_n
)
```

For comparing multiple pages, use `top_n` grouped by page:
```
fullstory:build_metric(
  query="console errors by page",
  output_type="top_n"
)
```

### Step 3: Add device/browser breakdown

If the user wants to know "mobile vs desktop on this page":
```
fullstory:build_metric(
  query="page views on /checkout by device type",
  output_type="top_n"
)
```

Or for errors specifically:
```
fullstory:build_metric(
  query="console errors on /checkout by device type",
  output_type="top_n"
)
```

### Step 4: Investigate problem pages

If a page stands out (high error rate, high abandonment):
1. `fullstory:get_sessions(page="/checkout", metric_id=error_metric_id, limit=5)`
2. Load sessions through `session-context` agent: "What error occurred on /checkout? What page did the user come from and go to?"
3. Synthesize: is the error specific to this page, or does it follow the user from a previous page?

### Step 5: Navigation patterns

To understand what pages users visit before/after a given page:
```
fullstory:get_user_pages(uid="...", options={...})
→ returns all pages visited by a user in order
```

But this is per-user. For aggregate navigation patterns, build a funnel metric with the page as a step and compute:
```
fullstory:build_metric(
  query="users who viewed /pricing, then what page did they view next",
  output_type="top_n"
)
```

## Common Analysis Patterns

| Question | Approach |
|----------|----------|
| "Which page has the most errors?" | `build_metric("console errors by page", "top_n")` → compute |
| "How does /checkout perform on mobile?" | `build_metric("errors on /checkout by device", "top_n")` |
| "What's the slowest page?" | `get_opportunities` for page-level rage clicks / abandonment |
| "What pages lead to /checkout?" | Scan sessions that visited /checkout, note previous page |
| "Page X vs Page Y error rate" | Two metrics, compute both, present side by side |
| "Has /settings gotten worse?" | Trend metric for errors on /settings, look for inflection |

## Guidelines

- Page names in Fullstory are URLs — use path patterns like `/checkout` or `/products/*`.
- When comparing pages, always normalize by traffic. A page with 10 errors and 100 views is worse than a page with 50 errors and 10,000 views.
- If a page has high error rate on a specific device/browser, flag it immediately — these are often easy-to-fix compatibility bugs.
- Navigation pattern analysis is fuzzy — Fullstory doesn't have a native "page flow" tool. Be transparent that you're inferring from individual sessions, not getting a computed flow diagram.
- Always surface the `metric_url` for any page-level metric you build.
