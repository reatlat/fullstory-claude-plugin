---
name: session-playlist
description: Curate playlists of Fullstory sessions for team review — bundle related sessions with notes, timestamps, and context. Use when sharing session evidence with the team, preparing for a bug review, or creating a highlight reel of user behavior.
---

# Session Playlist

Curate and share playlists of Fullstory sessions — bundle related sessions with notes, timestamps, and context so your team can watch exactly what matters, in order.

## When to Use

- "Bundle these 5 sessions of the checkout bug for the engineering team"
- "Create a playlist of power user sessions for the design review"
- "Share the sessions from the incident investigation"
- "Put together a highlight reel of the new feature adoption"
- "Send the PM 3 sessions showing why users abandon the form"

## Mental Model

A playlist is a curated set of sessions with context. Instead of sending 5 raw session links in Slack ("check out these sessions"), you send a playlist with:
- **Title**: What this playlist shows
- **Context**: Why these sessions matter
- **Per-session notes**: What to watch for at which timestamp
- **Order**: Narrative flow (e.g., "first bug → attempted workaround → successful flow")

## Workflow

### Step 1: Define the playlist

Ask the user:
- What's the theme? (bug evidence, user research, onboarding review, incident evidence)
- Who's the audience? (engineers, designers, PMs, execs)
- How many sessions? (3-5 is ideal — more than 10 and nobody watches them all)

### Step 2: Select sessions

Pull sessions relevant to the theme. Use relevant skills:
- `frustration-hunter` → sessions with rage clicks or dead clicks
- `error-forensics` → sessions with specific errors
- `session-search` → sessions matching specific criteria
- `customer-360` → sessions from a specific user
- `user-journey` → sessions across a user's timeline

For each candidate session, open it with `session_view` to find the key moment — the specific timestamp where the interesting thing happens.

### Step 3: Annotate each session

For each session in the playlist:
```
Session 1: Checkout bug — "Apply Promo" not working
• URL: https://app.fullstory.com/ui/org/session/abc123
• Key moment: 02:34 — user enters promo code "SAVE20"
• Watch for: Toast says "applied" at 02:36, but cart total doesn't change
• User reaction: Clicks "Apply" 5 more times, rage clicks at 02:45, abandons at 03:10
• Device: iPhone 15, Safari
```

### Step 4: Order for narrative

Arrange sessions to tell a story:
- **Bug playlist**: First occurrence → worst case → workaround attempt → edge case
- **User research**: New user → casual user → power user → churned user
- **Incident evidence**: Earliest affected session → peak incident → post-fix verification
- **Feature adoption**: First try → second session → regular usage → advanced usage

### Step 5: Write the playlist summary

```
## Playlist: Checkout Promo Code Bug — Evidence Package

**For**: Frontend engineering team
**Created**: Aug 4, 2026
**Context**: Rage clicks on "Apply Promo" button up 23% this week. These 5 sessions show the pattern.

### Sessions (watch in order)

1. **The canonical case** (2:34)
   🔗 [Session abc123](url)
   User enters "SAVE20", toast confirms, total unchanged. Rage clicks, abandons.
   → Shows the core bug clearly.

2. **Desktop variant** (1:12)
   🔗 [Session def456](url)
   Same behavior on Chrome desktop. Not mobile-only.
   → Confirms cross-platform.

3. **Multiple attempts** (5:45)
   🔗 [Session ghi789](url)
   User tries 3 different promo codes. All fail silently. Eventually checks out at full price.
   → Shows this isn't a one-off — all codes fail.

4. **The workaround** (3:20)
   🔗 [Session jkl012](url)
   User rage-clicks, then opens support chat. Agent applies discount manually. User completes purchase.
   → Support team is fielding this bug. Shows the manual fix cost.

5. **Post-incident verification** (0:45)
   🔗 [Session mno345](url)
   After the fix was deployed. Promo code applies correctly, total updates. No rage clicks.
   → Confirms the fix works.

### Key Takeaway
The bug is: toast fires optimistically before the API confirms the promo. Fix: await API confirmation before showing the toast. Affects all devices, all promo codes. Support team is manually applying discounts as a workaround.
```

### Step 6: Deliver

Present the playlist as a markdown document. Offer:
- "Copy this into a Slack thread, Notion doc, or Jira ticket."
- "Want me to add more sessions, or reorder these?"
- "Should I pull one more session showing an edge case?"

## Playlist Templates

### Bug Report Playlist
```
1. The bug (canonical case)
2. The bug on a different device/browser
3. The user's workaround or recovery
4. A session where the bug didn't occur (control case)
5. Post-fix verification (if fix is deployed)
```

### User Research Playlist
```
1. New user — first session, onboarding experience
2. Casual user — occasional usage, basic features
3. Power user — advanced features, high engagement
4. At-risk user — declining usage, frustration signals
5. Churned user — last sessions before leaving
```

### Design Review Playlist
```
1. Happy path — everything works as designed
2. Edge case — unusual input or flow
3. Mobile variant — same flow on a phone
4. Error state — what happens when things break
5. Accessibility — keyboard-only or screen reader session (if available)
```

## Guidelines

- 3-5 sessions max. Nobody watches 10 session replays. If you need more evidence, summarize: "The same pattern appears in 8 additional sessions — all showing the toast/cart disconnect."
- Timestamps are essential. "Watch at 2:34" is the difference between "I watched the session" and "I skimmed the session and missed the important part."
- Order matters. Don't dump sessions chronologically. Order for narrative: canonical → variants → edge cases → resolution.
- Include device/browser context. A bug that's Safari-only is a different engineering problem than a cross-platform bug.
- The playlist summary is the artifact. The user should be able to copy-paste the entire playlist into a Jira ticket or Slack thread without editing.
