---
name: query-translator
description: Translate vague business questions into precise Fullstory queries — help users who know what they want to ask but not how to express it. Use when the user says "I want to know X but I'm not sure how to ask Fullstory."
---

# Query Translator

Help users who know what they want to ask but not how to phrase it for Fullstory. Turn "I want to know if people like the new feature" into measurable, query-able questions.

## When to Use

- "I want to know if users are happy with the redesign — how do I ask that?"
- "What's the right way to measure engagement?"
- "I know what I want to learn but I don't know what to ask Fullstory"
- "Help me formulate a query"
- "Is there a way to measure X?"

The `query-translator` is invoked when the user's question is vague, abstract, or not directly answerable with Fullstory tools. It helps bridge the gap between "what I want to know" and "what Fullstory can measure."

## Mental Model

Most business questions are too vague to query directly. The skill of query translation is finding a measurable proxy:

| Vague Question | Measurable Proxy |
|---------------|-----------------|
| "Do users like the new feature?" | Adoption rate, repeat usage, rage clicks on the feature |
| "Is our product easy to use?" | Task completion rate, rage clicks, dead clicks, time-on-task |
| "Are users engaged?" | Session frequency, pages per session, features used, return rate |
| "Is the redesign better?" | Before/after: conversion, errors, rage clicks, scroll depth |
| "Are we losing users?" | Churn rate, at-risk user count, declining session frequency |
| "Is our support experience good?" | Sessions visiting help pages, search usage, rage clicks on help content |

## Workflow

### Step 1: Understand the real question

When the user asks something vague, dig deeper:

User: "I want to know if users like the new dashboard."
You: "That's a great question. Let me break that down. 'Like' could mean a few different things. Are you more interested in:
- Whether they're actually using it? (adoption metrics)
- Whether they're coming back to it? (retention metrics)  
- Whether they're struggling with it? (frustration metrics)
- How it compares to the old dashboard? (comparison metrics)"

### Step 2: Propose measurable proxies

Based on their answer, suggest specific metrics:

```
For "are they using it":
1. How many users visited the new dashboard? (awareness)
2. Of those, how many interacted with it? (trial rate)
3. Of those, how many used it again? (adoption rate)
4. How does this compare to the old dashboard's adoption? (benchmark)

For "are they struggling":
1. Rage clicks on the new dashboard elements
2. Dead clicks (trying to click things that aren't interactive)
3. Time spent on the page (confusion = longer time, or faster exit)
4. Errors on the new dashboard page
5. Users switching back to the old dashboard after trying the new one

For "is it better than the old one":
1. Task completion rate (new vs old)
2. Time to complete key tasks (new vs old)
3. Error rate (new vs old)
4. User satisfaction proxy: rage clicks + dead clicks (new vs old)
```

### Step 3: Prioritize

Help the user pick the 2-3 most important metrics:

"If you had to pick the most important signal, which would it be? Adoption rate (are they using it), frustration rate (are they struggling), or task completion (are they succeeding)?"

Most users pick one. Start there. You can always compute the others after.

### Step 4: Translate to Fullstory queries

Turn the chosen metrics into actual calls:

User chooses: "I want to know if they're using it and not struggling."

Translation:
```
1. Adoption: fullstory:build_metric(query="users who visited new dashboard in last 30 days", output_type="trend")
2. Frustration: fullstory:get_opportunities → filter by /dashboard/new → rage clicks and dead clicks
3. Comparison: fullstory:build_metric(query="old dashboard visits vs new dashboard visits", output_type="trend")
```

### Step 5: Execute and interpret

Run the queries and present results. Then close the loop:

"Here's what we found: 32% of users tried the new dashboard, and rage clicks are 40% lower than the old dashboard. That suggests users who try it find it less frustrating — which is a strong signal that they like it. The adoption rate is moderate — might be a discovery issue, not a quality issue."

### Step 6: Teach for next time

"So next time you want to know something like 'do users like X,' you can just ask:
- 'What's the adoption rate for X?'
- 'Are users rage-clicking on X?'
- 'Compare X usage to the old version'

Those three questions will give you a pretty complete picture."

## Common Translations

| User Says | Translate To |
|-----------|-------------|
| "Is our product sticky?" | Day 7 and Day 30 retention rate, session frequency distribution |
| "Are people finding what they need?" | Search usage rate, search → click-through rate, dead clicks on search results |
| "Is the onboarding working?" | Onboarding completion rate, time to complete, drop-off by step, new user activation rate |
| "Are we losing mobile users?" | Mobile vs desktop churn rate, mobile conversion trend, mobile rage click rate |
| "Is the pricing page effective?" | Scroll depth to pricing tiers, CTA click rate, pricing page → signup conversion |
| "Are users reading our content?" | Scroll depth, time on page, return visits, article completion rate |
| "Is our product fast enough?" | Page load frustration signals, rage clicks during load, bounce rate on slow pages |
| "Are users trusting us?" | Form abandonment at sensitive fields (payment, phone), privacy page visits, trust badge interaction |
| "Is our support experience good?" | Help center search success rate, rage clicks on help content, support ticket creation rate |

## Guidelines

- The user doesn't need to know the query syntax. Your job is to be the translator between "business question" and "Fullstory query." They speak business. You speak metrics.
- A vague question is an opportunity to teach. Walk through the translation so next time they can ask more precisely.
- If a question truly can't be answered with Fullstory data (e.g., "what's our NPS?"), say so and suggest the closest proxy: "Fullstory doesn't do surveys, but I can measure user frustration signals as a proxy for satisfaction."
- Don't overwhelm with 10 possible metrics. Pick the 2-3 best proxies and offer them as options.
- If the user pushes back on a proxy ("that doesn't really measure what I want"), adapt. They know their business better than you do.
