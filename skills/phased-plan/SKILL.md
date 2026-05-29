---
name: phased-plan
description: >
  Use when the user wants a structured, phased implementation plan with
  commit-level granularity. Trigger on: "phased plan", "break into phases",
  "step by step implementation", "incremental approach", or any request to
  plan a multi-part feature or refactor before writing code. Prefer this over
  ad-hoc planning whenever the scope involves 3+ logical changes or multiple
  files.
---

# Phased Plan Skill

## Core Behavior

Enter plan mode and produce a phased implementation plan in the form of a
markdown document. Each phase of the plan should map to exactly one logical
commit — cohesive, reviewable, and independently shippable where possible.
Favor more smaller phases over fewer large ones: they're easier to review,
allow course-correction between phases, and produce a cleaner git history.

## Avoid Broken Commits

Each commit must represent a functioning state of the program. When replacing
existing functionality, do not remove the old implementation before the new
implementation is integrated and wired in. Prefer "introduce → switch →
remove" rather than "remove → replace".

### Example:

**Bad:**

1. Rip out old CLI flag implementation (broken state)
2. Add new implementation (fixes previous commit)

**Good:**

1. Introduce new implementation alongside old (both exist)
2. Switch CLI flag to new implementation
3. Remove old implementation

## Input

If `$ARGUMENTS` is provided, treat it as the task description. If empty, ask
the user what they want to implement before proceeding.

## Before Planning: Explore First

Explore just enough to plan accurately — not the entire codebase:

1. Run `ls` / `find` and read the README to understand project shape.
2. Cap exploration at ~10 files; if scope is still unclear, ask the user to
   point to the relevant module rather than exploring further.
3. Identify natural seams where work can be split (data model, API, UI,
   tests, docs) and order phases so each builds cleanly on the last.

## Phase Format

Use this exact structure for every phase:

---

### Phase N — \<short title\>

**Goal:** One sentence describing what this phase accomplishes.

**Changes:**
- Bullet list of concrete changes (files, functions, configs, etc.)

**Success criteria:**
- [ ] \<specific, verifiable outcome\>
- [ ] \<specific, verifiable outcome\>

**Quality gates** — all must pass before the phase is complete.

Adapt to the project: remove gates for tooling that isn't configured, add
project-specific ones (type-check, E2E suite, etc.).

- [ ] Code compiles / runs without errors (if applicable)
- [ ] Linter / formatter passes (if configured)
- [ ] No new test failures vs. baseline (if a test suite exists)
- [ ] New tests pass (if this phase adds tests)
- [ ] Manual smoke test: \<describe what to verify\>

**Commit message:** `<type>: <concise description>`

---

End the plan with this footer (paste verbatim):

> After each phase, I will pause for your review before committing and moving
> to the next. To request changes, describe what to adjust and I will revise
> before asking again.

## Update Documentation

Always include an extra final phase for updating documentation, even if it's
just a README or inline code comments for non-trivial implementation. This
ensures docs stay up to date. Do not forget agent context files such as
`AGENTS.md` and `CLAUDE.md`.

## Execution (after plan approval)

Once the user approves the plan, run the full test suite once to record a
baseline of any pre-existing failures. This baseline is the comparison point
for every phase — the gate is no *new* failures, not necessarily a fully
green suite.

Then work through phases one at a time:

1. Implement the phase.
2. Run all quality gates. For the test gate, compare against the baseline.
   Fix any new failures before continuing; pre-existing failures are not
   your responsibility in this phase.
3. Verify every success criterion is met.
4. **Stop and present a summary** of what was done. Include your recommended
   commit message.
5. Wait for explicit approval ("approve", "proceed", "lgtm", or similar).
6. Only after approval: commit using the phase's commit message, then move on.

Never auto-advance to the next phase. Never commit without explicit approval.

## Handling Stuck Quality Gates

If quality gates can't be made to pass after reasonable effort, stop. Describe
what's failing and why, then ask the user how to proceed: skip the gate, debug
together, or revise the phase.

## Handling Mid-Plan Changes

If a phase reveals that a later phase needs adjustment:

- **Minor change** (one phase affected, clearly better path): propose the
  revision proactively before continuing.
- **Significant change** (multiple phases affected, or involves a design
  decision): stop and ask the user for guidance before proposing anything.

Never silently deviate from the approved plan.
