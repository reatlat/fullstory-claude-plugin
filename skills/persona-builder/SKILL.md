---
name: persona-builder
description: Build behavioral user personas from session data — identify distinct user types by their actual behavior, not demographics. Use when the team needs data-driven personas, wants to understand who their users really are, or needs to align product decisions to real user behavior.
---

# Persona Builder

Build behavioral personas from real session data — identify who your users actually are based on what they do, not what you assume.

## When to Use

- "What types of users do we have?"
- "Build personas for the product team"
- "Who are our power users vs casual users?"
- "What's the difference between users who convert and users who don't?"
- "Define our ideal customer profile from behavioral data"

## Mental Model

Traditional personas are built from interviews and assumptions. Behavioral personas are built from what users actually do. The difference matters:

- "Sarah the Marketing Manager, 34, Chicago" — traditional persona (demographic)
- "The Power Browser: visits 3+ times/week, compares products extensively, reads reviews, converts on 4th visit" — behavioral persona (what matters for product decisions)

## Workflow

### Step 1: Discover behavioral clusters

Run broad metrics to find natural grouping dimensions:
```
fullstory:build_metric(query="sessions per user in last 30 days", output_type="top_n")
fullstory:build_metric(query="pages per session", output_type="top_n")
fullstory:build_metric(query="time on site per session", output_type="top_n")
fullstory:build_metric(query="features used per user", output_type="top_n")
```

Look for natural breakpoints:
- Session count: 1 (one-and-done), 2-5 (occasional), 6-20 (regular), 20+ (power)
- Pages per session: 1-2 (bouncers), 3-5 (browsers), 6+ (deep explorers)
- Features: 1-2 (single-purpose), 3-5 (multi-feature), 6+ (power user)

### Step 2: Build segments for each cluster

```
fullstory:build_segment("users with 20+ sessions in last 30 days") → power_users
fullstory:build_segment("users with 1 session in last 30 days") → one_timers
fullstory:build_segment("users with 2-5 sessions, 3-5 pages per session") → casual_browsers
```

### Step 3: Profile each persona

For each segment, pull sessions and analyze:
```
fullstory:get_sessions(segment_id, limit=10)
→ session-context: "What is this user's typical flow? What features do they use? What frustrates them? What do they achieve?"
```

Build a profile for each persona:
- **Name**: Memorable label ("The Power Browser", "The Task Completer")
- **Behavior**: What do they do? (typical session flow)
- **Frequency**: How often? (sessions per week/month)
- **Goals**: What are they trying to accomplish?
- **Friction**: What frustrates them? What blockers do they hit?
- **Conversion**: Do they convert? What triggers conversion?
- **Size**: How many users are in this persona? (% of total)

### Step 4: Compare personas

Present side by side:
```
| Persona | Users | Sessions/Mo | Conv Rate | Top Frustration |
|---------|-------|-------------|-----------|-----------------|
| Power Browser | 1,200 (8%) | 12 | 32% | Slow search results |
| Task Completer | 4,500 (30%) | 3 | 45% | Checkout form on mobile |
| Casual Browser | 6,200 (41%) | 2 | 8% | Can't find pricing |
| One-Timer | 3,100 (21%) | 1 | 2% | Site loads slowly |
```

### Step 5: Generate persona cards

```
## Persona: The Power Browser
**1,200 users (8%)** | 12 sessions/month | 32% conversion

### Typical Session
Lands on homepage → searches for product → browses 5-8 product pages → reads 2-3 reviews → adds to cart → checks shipping → may or may not complete purchase

### What They Want
- Fast, accurate search (their top frustration is slow search)
- Detailed product specs and comparison
- Trust signals (reviews, ratings, shipping info)

### How to Serve Them
- Improve search performance (their #1 complaint)
- Add product comparison feature
- Surface reviews and ratings prominently

### What Converts Them
- Free shipping threshold met
- Product review rating >4.5 stars
- Competitor price comparison visible
```

### Step 6: Distribute

Format the personas for different audiences:
- **Product team**: Behavioral profiles with feature implications
- **Marketing**: Messaging angles per persona, channel preferences
- **Engineering**: Technical friction points per persona, performance priorities
- **Exec**: Persona mix trends, revenue per persona

## Persona Discovery Dimensions

| Dimension | What to Look For | Segments |
|-----------|-----------------|----------|
| Frequency | Sessions per month | Daily, Weekly, Monthly, Quarterly |
| Depth | Pages per session | Bouncers (1-2), Browsers (3-5), Explorers (6+) |
| Breadth | Features used | Single-purpose, Multi-feature, Power user |
| Intent | What they accomplish | Researchers, Buyers, Support-seekers, Window-shoppers |
| Journey stage | Where in lifecycle | New, Active, At-risk, Churned (see `lifecycle-analyzer`) |
| Device | Primary device | Desktop-first, Mobile-first, Cross-device |

## Guidelines

- Behavior trumps demographics. "34-year-old woman in Chicago" tells you nothing about what to build. "Compares 5+ products before purchasing" tells you everything.
- 3-5 personas max. More than that and they blur together. If you find 8 clusters, merge the smaller ones into the larger ones.
- Personas should be discoverable in the data, not invented. If you can't build a segment for a persona, it's not a persona — it's a guess.
- Update personas quarterly. User behavior changes as the product evolves.
- The "One-Timer" persona (1 session, never returned) is often the largest group. That's normal. Focus product decisions on the personas that generate value (Power Browsers, Task Completers).
