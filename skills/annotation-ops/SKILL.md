---
name: annotation-ops
description: Create and manage Fullstory timeline annotations — deploy markers, experiment start/end, incident windows, and release notes. Use when the user wants to mark a deploy, experiment, incident, or any notable event on the Fullstory timeline.
user-invocable: false
---

# Annotation Ops

Create and manage annotations on the Fullstory timeline so you can see how deploys, experiments, and incidents correlate with changes in user behavior.

## When This Runs

Invoked automatically when the user asks:
- "Mark the 3pm deploy on the timeline"
- "Add an annotation for the experiment we started yesterday"
- "Tag the incident window from 2-4pm today"
- "Show me annotatons from the last deploy"

The skill is `user-invocable: false` — it runs when `general-analysis` or `weekly-digest` references an annotation, or when the user explicitly asks to mark something.

## What Annotations Are

Annotations are labeled time ranges on the Fullstory timeline. They're markers you set so that when viewing metrics, you can see: "Oh, the conversion drop started right when we deployed the new checkout."

Common annotation types:
- **Deploy marker**: A point in time when code was released
- **Experiment start/end**: The window an A/B test ran
- **Incident window**: The time range of a production issue
- **Feature flag toggle**: When a feature was turned on or off
- **Marketing campaign**: When an email blast or promotion ran

## Workflow

### Step 1: Clarify the annotation

When the user says "mark the deploy" or "tag the incident", ask or infer:
- **Label**: What should it say? (e.g., "v2.3.0 deploy", "Checkout experiment start", "API incident")
- **Time**: When did it start? (e.g., "3pm today", "yesterday at 2pm", "July 15 10:00 UTC")
- **End time**: For events with duration (incidents, experiments) — when did it end? For point-in-time events (deploys) — no end time or use start == end.
- **Description** (optional): Any extra context — what changed, who deployed, incident ticket link.

### Step 2: Create the annotation

Call `fullstory:create_annotation`:
- `label`: Human-readable name
- `start_time`: ISO 8601 timestamp
- `end_time`: ISO 8601 timestamp (omit for point-in-time events)
- `description`: Optional context

The tool creates the annotation on the Fullstory timeline and returns an annotation ID. Confirm to the user: "Annotation 'v2.3.0 deploy' created at July 15 15:00 UTC."

### Step 3: Use in analysis

After creating an annotation, offer to check metrics around that time:
"You marked the deploy at 3pm. Want me to check if conversion changed after the deploy?"

When running `general-analysis` or `funnel-doctor` near an annotation time, reference it: "Checkout conversion dropped 4 points starting July 15th — coinciding with the 'v2.3.0 deploy' annotation."

## Annotation Best Practices

- **Be specific in labels**: "v2.3.0 deploy" not "deploy". "Checkout experiment v2" not "experiment".
- **Include the incident link** in the description for incident annotations.
- **Use point-in-time** (start == end) for deploys and feature toggles. Use **duration** for experiments and incidents.
- If the user mentions a time relative to now ("3pm today", "yesterday"), convert to the correct date — don't ask them to do it.

## Reading Annotations

If the user asks "what annotations exist around this time?", annotations are visible in the Fullstory UI on the timeline. The MCP server doesn't currently have a `list_annotations` tool — direct the user to the Fullstory app to see them visually alongside their metrics.
