# Doc Authoring Guidelines

These rules govern all documentation produced by the `agent-docs` skill. Follow them when generating or editing any agent doc.

## Length budgets

Files beyond ~200 lines consume disproportionate context and reduce adherence. Targets:
- Leaf / package `AGENTS.md`: ~150 lines
- Monorepo root `AGENTS.md` covering several services: up to ~250 lines

## Standard section order

```
# AGENTS.md — <Package Name>
## Purpose           (1-3 sentences)
## Architecture      (if multi-component)
## Key Files         (table)
## Build & Test      (exact commands)
## Code Conventions  (concrete rules)
## Critical Gotchas  (numbered, worst first)
## Terminology       (domain terms agents won't know)
## Do
## Don't
```

Skip sections that don't apply. Never add sections not on this list without a clear reason.

## Structural rules

1. **No prose paragraphs** — tables, bullet lists, code blocks. Agents parse structure, not essays.
2. **Key Files table** columns: `File | Lines (approx) | Purpose (one sentence)`. Use `~` prefix for counts (`~2.4k`).
3. **Source Layout trees** — ASCII `├──` format with brief annotations.
4. **Code Conventions** — concrete rules with `CORRECT ✓` and `WRONG ✗` examples.
5. **Test commands** — exact, copy-pasteable. Never "run the tests" without specifying how.

## Content rules

1. **Reference actual paths** — always backtick-quoted, relative to repo root. Verify with `ls` before writing.
2. **Critical Gotchas first** — lead with mistakes agents WILL make without guidance. Highest-ROI content.
3. **Concrete enough to verify** — every instruction must be checkable: "run `npm test` before committing" not "test your changes".
4. **No auto-generated comments** — docs read as natural, authoritative references by a knowledgeable engineer.
5. **No contradictions across the hierarchy** — when two rules conflict (root vs nested `AGENTS.md`), resolve in favor of the more specific (nearest) doc. Delete or scope the loser.

## Stack-specific rules

Derive from the repo itself — never apply a generic per-language rulebook. For each language/framework present, extract and document:
- Compiler/linter strictness (flags that turn warnings into errors, naming rules enforced by tooling)
- Generated-file policies (what must never be hand-edited, what command regenerates it)
- Framework conventions (where each kind of file lives, routing/registration steps easy to miss)
- Expected environment variables and version floors

Source: linter/formatter configs, CI config, existing docs. Never from assumption.

## Regeneration rules

1. **Surgical edits** — don't rewrite an entire doc for one stale reference.
2. **Full rewrite threshold** — if >40% of a doc's file references are broken, regenerate from scratch.
3. **Preserve human voice** — if a doc has human-authored architecture rationale or design decisions, keep those sections. Only update factual references.
4. **Line count updates** — use `wc -l`. Round to nearest 50 for files <1000 lines, nearest 100 for larger. Use `~` prefix.
