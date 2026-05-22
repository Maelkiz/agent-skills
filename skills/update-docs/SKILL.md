---
name: update-docs
description: >
  Use when the user wants to ensure documentation is up to date, including but not limited to docs/, README, inline code comments, or agent context files such as `AGENTS.md` and `CLAUDE.md`.
---

# Update Docs Skill

## Overview

Documentation drift is addressed in three phases: **Inventory**, **Audit**, and **Execution**. Each phase builds on the last. Never skip ahead — fixes made before the full audit is done will miss things and produce noisy commits.

If `$ARGUMENTS` is provided, treat it as a scope hint (e.g. "README only", "CLAUDE.md and inline comments"). If empty, treat the scope as all documentation in the project.

---

## Phase 1 — Inventory

Two goals: locate all documentation, and map the code structures those docs make claims about.

### 1a. Find documentation files

Search for:
- Top-level prose: `README*`, `CHANGELOG*`, `CONTRIBUTING*`, `ARCHITECTURE*`, `docs/`
- Agent context files: `CLAUDE.md`, `AGENTS.md`, `.cursorrules` (any directory)
- Inline comments: docstrings, JSDoc, rustdoc — skim entry points and public APIs
- Config documentation: comments inside `Makefile`, `docker-compose.yml`, CI workflows that explain usage
- Example files: `examples/`, `*.example.*`, embedded code blocks in markdown

### 1b. Map code ground truth

For each documentation area, identify the authoritative code source:
- Entry points and public API surface (exports, CLI flags, HTTP routes)
- Key data shapes (types, schemas, config options)
- How-to-run instructions → `package.json` scripts, `Makefile` targets, or equivalent
- Architecture claims → directory structure, module boundaries

Cap exploration at ~15 files. If scope is too broad to map confidently, ask the user to narrow it.

### Phase 1 output

Produce a two-column manifest:

| Doc file / location | Claims to verify |
|---|---|
| `README.md` → Installation section | `npm install`, Node version requirement |
| `CLAUDE.md` → Commands section | `npm test`, `npm run build` |
| `src/auth/index.ts` docstring | `login(email, password)` signature |
| ... | ... |

Present this manifest to the user. If scope is larger than expected, ask whether to narrow it before continuing.

---

## Phase 2 — Audit

Work through every row in the manifest. For each claim, verify it against the current code and classify the finding.

### Claim types and how to verify each

| Claim type | How to verify |
|---|---|
| **Function/API signature** | Read the actual source; compare name, params, return type |
| **Behavior description** | Trace the code path; confirm the described behavior matches |
| **Example code** | Run or compile it if possible; otherwise manually trace it |
| **How-to-run instruction** | Check the script/target exists; run it if safe to do so |
| **File path reference** | Confirm the path exists |
| **Cross-reference / link** | Confirm the anchor, heading, or URL resolves |
| **Architecture claim** | Verify against directory structure and module imports |
| **Config option** | Check the schema or parser |

### Findings classification

For every discrepancy, record:

- **Stale** — documented something that no longer exists or has changed
- **Missing** — code has something real and useful that docs omit
- **Inaccurate** — claim is wrong but the documented thing still exists
- **Broken example** — code block that doesn't run or compile
- **Broken reference** — dead link, missing file, bad anchor

Accurate, up-to-date claims require no entry.

### Phase 2 output

Present a structured findings report before doing any editing:

```
FINDINGS
========

README.md — Installation
  [Stale]   Node >=16 required; package.json engines field says >=20
  [Missing] No mention of the required GITHUB_TOKEN env var

CLAUDE.md — Commands
  [Inaccurate] `npm run build` is now `npm run compile`

src/auth/index.ts docstring
  [Stale]   login() no longer accepts a password param; uses OAuth now
  [Broken example] Code block calls login("user", "pass") — wrong signature
```

**Stop here.** Ask the user to confirm scope before proceeding:
- Are there findings they want to skip or defer?
- Are any files off-limits for automated edits (e.g. a manually curated CHANGELOG)?
- Any findings that should be tracked as issues rather than fixed now?

Only proceed to Phase 3 after explicit confirmation.

---

## Phase 3 — Execution

Invoke the `phased-plan` skill, passing the confirmed findings as the task description. Each logical group of related fixes becomes one phase and one commit.

### How to group findings into phases

Group by documentation area and audience, not by file. Examples:
- All README fixes together (one commit, one PR-reviewable unit)
- All CLAUDE.md / AGENTS.md fixes together (agent context is a coherent unit)
- All inline docstring fixes per module (one commit per module)
- Broken examples as their own phase (needs verification, distinct risk)

Avoid mixing unrelated doc areas in one commit — a CLAUDE.md update and a README update should be separate even if both are small.

### Quality gates for documentation phases

Replace phased-plan's code-centric gates with these:

- [ ] Every fixed claim verified against current code (re-read the source, don't rely on memory)
- [ ] Any updated code examples run or compile without error (or are explicitly marked as pseudocode)
- [ ] All file paths and cross-references in the changed section resolve
- [ ] No previously accurate claim was accidentally altered
- [ ] Prose is consistent in tense, voice, and terminology with surrounding doc

### Commit message convention

```
docs(<area>): <concise description of what changed>
```

Examples:
- `docs(readme): update Node version requirement and add GITHUB_TOKEN setup`
- `docs(claude.md): correct build command and remove stale test instructions`
- `docs(auth): update login() docstring to reflect OAuth migration`

---

## Handling Scope Creep

If Phase 2 reveals documentation so far out of date that fixing it requires understanding code changes that aren't yet committed (i.e., the code itself is the source of truth but is also in flux), stop and tell the user. Don't guess at intended behavior.

If a doc fix would require changing the code (e.g., an example that's broken because the API is poorly designed), flag it as a separate issue rather than working around it in the docs.
