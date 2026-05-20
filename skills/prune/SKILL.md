---
name: prune
description: >
  Surgically cleans a live Claude Code session by removing high-token dead weight
  (absorbed tool output, completed sub-tasks, abandoned approaches) while
  explicitly preserving what compaction routinely loses: the *why* behind
  decisions, rejected alternatives and their reasons, active constraints, and
  open work. Use /prune when the session feels bloated but you're still mid-task
  and don't want to lose your place. More targeted than /compact (which
  summarises everything indiscriminately) and less destructive than /clear
  (which wipes everything). Also useful to run *before* /compact so the
  compaction summary is built from a cleaner signal.
disable-model-invocation: true
---

# /prune — Surgical Context Cleaning

## Why this exists

`/compact` optimises for "what to do next." It reliably drops:
- *Why* approach A was rejected in favour of approach B
- Constraints and gotchas surfaced mid-session ("don't touch this file")
- Subtle style rules established early ("no emoji," "no default exports")
- Specific reasoning chains behind architectural decisions

`/prune` runs *before* that information is lost — or before you run `/compact` —
so those high-value signals survive. It also removes the true bulk of token
waste: large tool outputs that have already been acted on.

---

## Step 1 — Identify what to remove (dead weight)

Scan the conversation for these categories. They are safe to discard:

**A. Absorbed tool output** — file reads, grep results, bash output, test logs
that you have already used to make a decision. These are typically 60–80% of
total token usage. Replace each with a one-line note of what was learned.
Example: `read_file src/auth.ts → expiry check missing on line 47`

**B. Completed sub-tasks** — work that is fully done and no longer referenced.
A fixed bug, a written file, a passing test suite. Keep only the outcome as a
single line. Example: `auth.ts + middleware.ts patched; all tests passing`

**C. Abandoned approaches** — explored paths that were ruled out. Keep the
*conclusion* only. Example: `JWT refresh rejected — breaks existing sessions`

**D. Repeated context** — the same information restated across multiple turns.
Keep the most recent version, remove earlier repetitions.

**E. Exploration scaffolding** — early codebase orientation messages once the
relevant parts are already known and have been acted on.

---

## Step 2 — Identify what to keep (high-value signals)

Never remove these — they are what compaction loses and what makes the
difference between a sharp session and a drifting one:

**Decisions + reasoning** — not just *what* was decided, but *why*. This is
the single most important thing to preserve. Format as:
`Decision: [what] — Reason: [why] — Rejected: [alternatives]`

**Active constraints** — anything the user has said must or must not happen.
These are typically stated once early and then forgotten by compaction.
Examples: "don't modify migrations," "backward compatibility required," "no new
npm packages"

**Open work** — current task, incomplete sub-tasks, unresolved errors

**Active files** — files currently being edited or that will be needed soon

**User preferences established in-session** — naming conventions, style rules,
format preferences that aren't in CLAUDE.md

---

## Step 3 — Produce the pruned context block

Write a replacement block in this format. Keep it tight — every line here
persists in context for the rest of the session:

```
## Session Context (pruned)

**Task:** <one sentence>

**Open sub-tasks:**
- <item> OR "none"

**Decisions:**
- <what> — <why> — rejected: <alternatives> (if any)

**Active constraints:**
- <constraint>

**Active files:** <comma-separated list>

**In-session preferences:** <style/format rules not in CLAUDE.md> OR "none"

**Dead-end log:** <one-line summary of each ruled-out approach>
```

---

## Step 4 — Report and continue

State in one line what was removed and approximately how much was cleared.
Example: `Pruned 4 file reads, 2 completed sub-tasks, 1 test log. ~8k tokens
freed. Decisions and constraints preserved above.`

Then continue the session from the pruned context.

---

## Arguments

If `$ARGUMENTS` is provided, treat it as a hint about what to protect.
Examples:
- `/prune keep the auth decisions` → anchor the decisions block around auth
- `/prune task complete` → treat the current task as done; keep only the
  dead-end log and constraints for the *next* task

---

## Notes

- If the session is clean and there is nothing worth pruning, say so in one
  line and stop. Do not pad.
- Do not re-read files to produce summaries — work only from what is already
  in context.
- This skill runs inline. No fork, no subagent.
- For maximum effect, run `/prune` first, then `/compact` — the compaction
  summary will be built from cleaner signal and the decisions block will
  survive it.