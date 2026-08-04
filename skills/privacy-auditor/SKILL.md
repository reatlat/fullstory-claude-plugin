---
name: privacy-auditor
description: Audit Fullstory session capture for PII leaks, verify privacy rules, and check recording block rules. Use when the user is concerned about privacy compliance, wants to verify masking rules, or needs to find sessions that may contain sensitive data.
---

# Privacy Auditor

Audit your Fullstory session capture for PII exposure, verify privacy rules are working, and find sessions that may contain sensitive user data.

## When to Use

- "Are we accidentally capturing PII in sessions?"
- "Check if our privacy masking rules are working"
- "Did the recent deploy break any privacy exclusions?"
- "Find sessions where user emails might be visible"
- "Verify that the /admin pages are excluded from recording"
- "Audit our recording block rules — are the right elements masked?"

## Mental Model

Fullstory records everything users see and do. Privacy controls prevent sensitive data from being captured:

1. **Exclusion rules**: Pages or elements excluded entirely from recording
2. **Masking rules**: Elements that are recorded but with text/images replaced
3. **Block rules**: Elements blocked from capture (not recorded at all)
4. **Network exclusions**: API calls excluded from network capture

This skill verifies that these controls are working correctly.

## Workflow

### Step 1: Check current privacy configuration

Call `fullstory:get_recording_block_rules` to see what's currently configured:
- Element selectors being masked or blocked
- URL patterns being excluded from recording
- Network URL patterns excluded from capture

Present the current rules to the user: "Here's what's currently configured. 12 element selectors masked, 3 URL patterns excluded, 1 network exclusion."

### Step 2: Verify rules on actual sessions

Pull recent sessions and check if the rules are working:

```
fullstory:get_sessions(time_range="last_24_hours", limit=5)
```

For each session, use `session_view` to check pages that should have masked elements:
- Navigate to a page with known sensitive elements (e.g., /checkout, /settings)
- Check that password fields, credit card inputs, and other sensitive fields are masked
- Verify that excluded pages (e.g., /admin) don't appear in session data at all

### Step 3: Hunt for PII leaks

Look for common PII exposure patterns:

**Fields that should be masked but might not be:**
- Email addresses in form fields, headers, or user profile displays
- Phone numbers in contact forms or account pages
- Physical addresses in shipping or billing forms
- Credit card numbers (should never appear — always masked by default)
- SSN, driver's license, passport numbers in verification forms

**To check**: Build a metric for sessions that visited pages with sensitive fields, then spot-check a few sessions with `session_view`:
```
fullstory:build_segment("users who visited /checkout or /account/settings in last 7 days")
fullstory:get_sessions(segment_id, limit=5)
→ session_view on the sensitive page, check for unmasked PII
```

### Step 4: Check for unintended recording

Pages that should NOT be recorded:
- Admin panels
- Internal tools
- Employee dashboards
- Authentication pages (login forms — should show as masked)

Build a segment for these pages and verify they're either excluded or properly masked:
```
fullstory:build_segment("users who visited /admin or /internal")
→ if sessions exist for these pages, check if they're properly excluded
```

### Step 5: Report

```
## Privacy Audit

### Current Rules
- 12 element selectors masked: ✅ Active
- 3 URL exclusions: ✅ Active (blocks /admin/*, /internal/*, /api/keys/*)
- 1 network exclusion: ✅ Active (blocks /api/payment-methods)

### Rule Verification
- Checked 5 sessions — all masked fields working correctly ✅
- Password fields masked in all sessions ✅
- Credit card fields masked in checkout ✅
- /admin pages: 0 sessions recorded (exclusion working) ✅

### PII Scan
- No unmasked emails, phone numbers, or addresses found in spot-checked sessions ✅
- One concern: /settings/profile shows user's full name in the header — this is unmasked. Is that intentional? 🟡

### Recommendations
1. Verify if full name display in /settings/profile should be masked
2. Consider adding /employee/* to URL exclusions if not already covered
3. Review privacy rules quarterly — last audit was 3 months ago
```

## Privacy Best Practices

| Rule | Recommendation |
|------|---------------|
| Password fields | Always masked by default — `input[type="password"]` |
| Credit card fields | Always block — `#card-number, [data-sensitive="payment"]` |
| Email displays | Mask in headers and account pages — `.user-email, .profile-email` |
| Admin pages | Exclude entirely from recording — `/admin/*, /internal/*` |
| API keys/secrets | Never record — exclude network calls to `/api/keys/*` |
| Customer support chat | Consider masking — agents may paste user PII |

## Guidelines

- This skill doesn't guarantee compliance with GDPR, CCPA, or other regulations. It's a technical audit, not a legal one. Always say so.
- If you find unmasked PII, flag it immediately with the specific session URL and the element that's exposed. This is a P0 privacy issue.
- Privacy rules can break after deploys — if the markup changes (e.g., CSS class renamed), the mask rule stops matching. After a major deploy, re-audit.
- Don't read session transcripts with potentially exposed PII into the main context. Use `session-context` agent with a task like "check if any text visible on /checkout looks like an email address or credit card number."
- Fullstory masks credit card numbers by default (PCI compliance). If you see one in a session, it's a bug in Fullstory's masking, not your rules — report it to Fullstory support.
