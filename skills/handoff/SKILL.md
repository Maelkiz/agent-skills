---
name: handoff
description: >
  Compact the current conversation into a handoff document for another agent to pick up.
argument-hint: "What will the next session be used for?"
---

# Handoff Skill

Creates a structured handoff document so a fresh agent can continue the work with a fresh context window, with only high quality, relevant information passed along.

Use this instead of `/compact` when you want a clean slate: a new session inherits
only what matters, rather than a compressed version of everything.

---

## How Handoffs Work

The handoff document is a self-contained briefing saved to the OS temp directory.
A fresh agent reads it at the start of the next session to get up to speed.

---

## Workflow

### Step 1: Assess the current session

Before writing, orient yourself on:

- What is the active goal and what stage is it at?
- What decisions were made that must carry forward?
- What work is complete vs still in progress?
- What blockers or open questions remain?
- Which files, modules, or tools are central to the work?

### Step 2: Write the handoff document

Use this template:

```markdown
# Handoff: <short description of the work>

## Context
<1–3 sentences: what problem is being solved and why it matters>

## Current Goal
<The specific task or milestone in progress right now>

## Work Completed
<Bullet list of what's done (reference artifacts by path/URL)>

## Key Decisions Made
<Bullet list of decisions that the next agent needs to know>

## In Progress
<What was actively being worked on when this session ended>

## Open Questions / Blockers
<Any unresolved issues, decisions needed, or things to watch out for>

## Next Steps
<Ordered list of what the next agent should do, starting with the very first action>

## Key References
<Paths and URLs the next agent will need>
```

Fill in each section based on what you learned in Step 1. Omit sections that
aren't relevant (e.g. no blockers → drop that section).

If the user passed arguments, treat them as a description of what the next session
will focus on and tailor the document accordingly — emphasize next steps and context
relevant to that focus.

### Step 3: Save and report

Save the document to the OS temp directory (not the workspace). Name it
`handoff-<slug>.md` where `<slug>` is a short kebab-case label for the work.

Report the full path so the user can pass it to a fresh agent:

> Handoff saved to `/tmp/handoff-<slug>.md`.
> Start the next session with: "Read /tmp/handoff-<slug>.md and continue from there."

---

## What to Omit

- Exploratory back-and-forth that led nowhere
- Dead-ends and false starts that won't help the next agent
- Any information that isn't actionable or relevant to the next steps
- Sensitive information: API keys, passwords, PII — redact these
