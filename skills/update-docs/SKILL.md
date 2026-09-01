---

name: update-docs

description: >
  Use when the user wants to ensure documentation is up to date, including
  but not limited to docs/, README files, inline code comments, or agent
  context files such as `AGENTS.md` and `CLAUDE.md`.

---

# Update Docs Skill

## Overview

Documentation drift is addressed in three phases: **Inventory**, **Audit**, and **Execution**. Each phase builds on the last.

Two rules the whole workflow hangs on:

- **Scope by evidence, not by reading everything.** Deterministic signals (broken references, docs older than the code they describe) build the worklist. Reading source files speculatively burns context before the first fix.
- **Never write a claim you didn't verify.** Every command, path, flag, default, and example you write or keep must be checked against the current code. The failure mode of doc syncing is replacing old guesses with new ones.

Do not make documentation changes before completing the audit. Editing while auditing loses the cross-doc picture and produces contradictory fixes.

If `$ARGUMENTS` is provided, treat it as a scope hint (e.g. "README only", "CLAUDE.md and inline comments"). If empty, treat the scope as all relevant documentation in the project.

---

## Phase 1 — Inventory

Locate relevant documentation and build a worklist of what to audit.

### 1a. Find documentation

Look for:

- Top-level docs: `README*`, `CHANGELOG*`, `CONTRIBUTING*`, `ARCHITECTURE*`
- Documentation directories: `docs/`
- Markdown and other doc files: `*.md`, `*.mdx`, `*.rst`
- Agent context files: `CLAUDE.md`, `AGENTS.md`, `.cursorrules` at any directory level
- Inline documentation: docstrings, JSDoc, rustdoc, and comments describing public APIs, behaviour, invariants, configuration, or constraints
- Configuration documentation: comments in `Makefile`, CI workflows, Docker files that explain usage
- Examples: `examples/`, `*.example.*`, executable code blocks in docs

Do not treat every code comment as documentation. Ignore trivial comments that merely restate what the code obviously does. Respect the requested scope.

### 1b. Prioritize by staleness signals

Two fast signals flag the highest-risk docs without reading everything:

**Broken references** — dead links, missing file paths, invalid anchors. Check with:
```bash
grep -rn '\[.*\](.*\.md)' <doc-file>  # internal links
grep -rn '`[^`]\+\.[a-z]\+`' <doc-file> | tr -d '`'  # backtick file refs
```

**Age gap** — a doc modified before the code it references is a suspect:
```bash
git log --oneline -1 --format="%ar" -- <doc-file>      # doc's last change
git log --oneline -1 --format="%ar" -- <source-file>   # code's last change
```

Docs with broken references or a significant age gap go to the top of the worklist.

### 1c. Map code ground truth

For each doc in the worklist, identify the authoritative source for its claims:

- Public API claims → actual exported functions, classes, types, routes, CLI definitions
- How-to-run → package scripts, `Makefile` targets, build configuration
- Dependency requirements → package manifests and lockfiles
- Configuration options → schemas, parsers, defaults, validation logic
- Architecture claims → directory structure, module boundaries, imports
- Behaviour descriptions → implementation and relevant tests

### Phase 1 output

Record findings in a ledger at `/tmp/docs-findings.md` — one section per doc. All findings from Phase 2 go here before any edits happen. The ledger survives context compaction.

```markdown
# Doc Findings

## README.md
...

## CLAUDE.md
...
```

---

## Phase 2 — Audit

Work through every doc in the worklist. Verify claims against current project state. Do not rely on memory or assumptions when a claim can be checked directly.

### Claim types and how to verify them

| Claim type | How to verify |
|---|---|
| **Function/API signature** | Read the actual source; compare name, parameters, return type |
| **Behaviour description** | Trace the relevant code path; consult tests |
| **Example code** | Run, compile, or validate against current API |
| **How-to-run instruction** | Confirm command exists; run it when safe and cheap |
| **Dependency/version** | Check manifests, lockfiles, toolchain config |
| **File path reference** | Confirm path exists (`ls`) |
| **Cross-reference/link** | Confirm target resolves |
| **Architecture claim** | Verify against current structure and module relationships |
| **Configuration option** | Check schema, parser, defaults, validation |

### Findings classification

For every discrepancy, add to the ledger under one of:

- **Stale** — documented thing still exists but has materially changed (wrong signature, renamed flag, outdated example)
- **Cruft** — documented thing no longer exists; the content should be removed
- **Missing** — important behaviour, API, or requirement exists but is absent where a reader would expect it
- **Broken reference** — dead link, missing file, invalid anchor
- **Unverified** — claim cannot be confidently verified within reasonable scope

Accurate claims require no finding. Do not manufacture problems because something is undocumented — documentation should be proportional to the project's needs.

**Code suspects:** when doc and code disagree and the code looks like the bug (doc describes intended behaviour), flag it rather than silently rewriting the doc to canonize the accident. Note it in the ledger under `[Code suspect]`.

### Phase 2 output

Structured findings report in the ledger:

```text
FINDINGS
========

README.md — Installation

  [Stale]           Node >=16 required; package.json says >=20
  [Missing]         Required GITHUB_TOKEN env var is not documented

CLAUDE.md — Commands

  [Inaccurate]      `npm run build` is now `npm run compile`

src/auth/index.ts — docstring

  [Cruft]           login() no longer accepts a password parameter
  [Broken reference] Link to /docs/auth.md — file deleted
```

---

## Phase 3 — Execution

Fix docs **one at a time, foundational docs first** (architecture/reference before READMEs and guides that cite them, so downstream fixes can rely on upstream ones).

- **Stale** → correct in place, re-verifying the replacement claim against code (not from memory of Phase 2)
- **Cruft** → delete the content. For ADRs/design docs, annotate "Superseded by …" and leave — they are records of decisions, not living docs
- **Missing** → write the content proportionally; a new config knob wants a table row, not an essay
- **Broken reference** → find where the target moved (`git log --follow`) and update; if deleted and no successor, remove the reference

**Editing standards:**

- **Minimal diffs** — don't reflow, reformat, or re-voice text whose content isn't changing; review noise buries real fixes
- **Match voice** — a refresh should be invisible except where it's a correction
- **No placeholders** — never commit TODO/TBD; document fully or record as remaining gap in the report
- **Never hand-edit generated docs** — fix the source or generator config and regenerate; note the generator in the report
- **No fabricated changelog entries** — backfill only from `git log` evidence, in the file's existing format

---

## Phase 4 — Validate and commit

**Cross-doc consistency sweep** — re-read every doc this run touched, checking the set: same feature named the same way everywhere; no doc still cites content this run moved or removed; no introduced contradictions.

**Report remaining gaps** — anything found but not fixable without human input, and any code suspects flagged during audit.

Stage only the doc files this run touched:

```bash
git add <docs touched>
git commit -m "docs: update documentation — <summary>

<Stale: X fixed | Cruft: Y removed | Missing: Z filled | Remaining: N>
"
```
