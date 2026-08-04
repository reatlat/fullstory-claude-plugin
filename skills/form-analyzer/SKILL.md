---
name: form-analyzer
description: Form field-level analysis — abandonment by field, validation errors, time-to-complete, and which fields cause users to leave. Use when investigating form performance, signup drop-off, or checkout form friction.
---

# Form Analyzer

Field-by-field form analysis — which fields cause abandonment, where validation errors hit, how long each field takes, and where users get stuck.

## When to Use

- "Which field on the signup form has the highest abandonment?"
- "Why are users dropping off at the password field?"
- "How long does it take users to complete the checkout form?"
- "Which form fields trigger the most validation errors?"
- "Compare form completion on mobile vs desktop"
- "Did the new inline validation reduce form abandonment?"

## Mental Model

A form is a sequence of fields users must fill. Each field is a micro-funnel: enter → validate → proceed. The goal is to find which fields cause users to stall, error, or abandon.

Forms have unique failure modes:
- **Confusion**: Users don't know what to enter (e.g., "Username" vs "Email")
- **Validation friction**: Strict rules reject valid input ("Password must contain a special character")
- **Technical failure**: Field doesn't respond, autofill breaks, input type mismatch
- **Trust barrier**: Asking for too much info too early ("Why do you need my phone number?")

## Workflow

### Step 1: Identify the form

Clarify which form and what success looks like:
- Form page: `/signup`, `/checkout`, `/contact`
- Success event: page navigation to `/signup/confirmation`, custom event `form_submitted`, or purchase completion
- Fields of interest: all fields, or specific ones the user suspects

### Step 2: Build form metrics

Start with the overall funnel:
```
fullstory:build_metric(
  query="users who visited /signup, then completed signup",
  output_type="funnel"
)
```

Then build field-level metrics. For each field, build a metric for users who interacted with the field but abandoned before completing the form:
```
fullstory:build_metric(
  query="users who focused or typed in the password field on /signup but did not reach /signup/confirmation",
  output_type="single_number"
)
```

### Step 3: Detect friction per field

For the highest-abandonment fields, investigate:

**Rage clicks on the field:**
```
fullstory:get_opportunities → filter to the form page → look for rage clicks on form elements
```

**Dead clicks on submit buttons:**
```
fullstory:build_metric(
  query="dead clicks on submit button on /signup",
  output_type="single_number"
)
→ if high, the button is unresponsive or validation is blocking silently
```

**Validation errors:**
Build metrics for console errors or custom validation-error events on the form page.

### Step 4: Time-to-complete analysis

```
fullstory:build_metric(
  query="time spent on /signup page",
  output_type="single_number"
)
```

Break down by completion vs abandonment:
- Users who completed: 2m 30s average
- Users who abandoned: 4m 15s average (spent longer before giving up)

Long time + abandonment = user struggling, not just distracted.

### Step 5: Read sessions for the worst fields

For the field with the highest abandonment:
```
fullstory:get_sessions(page="/signup", metric_id=field_abandonment_metric, limit=5)
→ session-context agent: "Did the user interact with the password field? What happened — did they get a validation error, leave the field blank, or type something and then delete it?"
```

### Step 6: Report

```
## Form Analysis — /signup (last 30 days, 12,000 visits)
Completion rate: 64%

### Field Abandonment
| Field | Users who abandoned after this field | % of total visitors |
|-------|--------------------------------------|---------------------|
| Email | 340 | 2.8% |
| Password | 1,240 | 10.3% ← WORST |
| Name | 210 | 1.8% |
| Submit button | 420 | 3.5% |

### Password Field Deep Dive
- 1,240 users (10.3%) abandoned at or after the password field
- Average time on the field before abandoning: 22 seconds
- 73% received a validation error ("Password must contain...")
- Rage clicks: 87 users rage-clicked the password field or submit button

### Session Evidence
In 4 of 5 sessions, users received "Password must contain at least one special character" — typed a special character — still got rejected (the character wasn't recognized). Users tried 2-3 times, then left.

### Recommendation
The special character validation is too strict. Common characters like ! and @ are rejected. Relax the regex or switch to a strength meter without hard requirements.
```

## Common Form Patterns

| Symptom | Likely Cause | Check |
|---------|-------------|-------|
| High abandonment on first field | Users weren't ready to commit — the CTA promised something else | Session transcripts: what page did they come from? |
| Abandonment on phone number field | Privacy concern ("why do you need this?") | Is the field required? Can it be optional? |
| Rage clicks on submit | Button disabled by validation, but validation message isn't visible | session_view the form at submit time — is the error visible? |
| Long time on one field | Confusing input format (date picker, phone format) | Check if autofill works. Add placeholder or format hint. |
| Abandonment spikes after deploy | Validation rules changed | Compare before/after deploy with `deploy-radar` |

## Guidelines

- Form analysis is hypothesis-driven. Don't just dump field stats — explain *why* a field is problematic, backed by session evidence.
- A field with high abandonment might be the symptom, not the cause. Users might abandon at the password field because the email field confused them and they were already frustrated.
- Time-on-field is a proxy for struggle. Short time + abandonment = likely a trust/privacy issue. Long time + abandonment = likely a usability issue.
- Mobile forms have unique issues: keyboard covering the field, autofill not working, touch targets too small. Always check device breakdown.
- If form conversion changed after a deploy, cross-reference with `deploy-radar` and `experiment-analyzer`.