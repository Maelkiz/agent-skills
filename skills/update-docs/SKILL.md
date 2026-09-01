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

Do not make documentation changes before completing the audit. This prevents incomplete exploration from producing noisy or contradictory edits.

If `$ARGUMENTS` is provided, treat it as a scope hint (e.g. "README only", "CLAUDE.md and inline comments"). If empty, treat the scope as all relevant documentation in the project.

---

## Phase 1 — Inventory

The goal is to locate relevant documentation and identify the authoritative sources of truth for the claims those documents make.

### 1a. Find documentation

Look for:

- Top-level documentation: `README*`, `CHANGELOG*`, `CONTRIBUTING*`, `ARCHITECTURE*`
- Documentation directories: `docs/`
- Markdown and other documentation files: `*.md`, `*.mdx`, `*.rst`
- Agent context files: `CLAUDE.md`, `AGENTS.md`, `.cursorrules` and equivalent files at any relevant directory level
- Inline documentation: docstrings, JSDoc, rustdoc, and comments describing public APIs, behaviour, invariants, algorithms, configuration, or constraints
- Configuration documentation: comments in `Makefile`, CI workflows, Docker files, configuration files, etc. that explain usage or behaviour
- Examples: `examples/`, `*.example.*`, and executable code blocks embedded in documentation
- Documentation referenced by other documentation

Do not treat every code comment as documentation. Ignore trivial comments that merely restate what the code obviously does.

Respect the requested scope. Do not expand into unrelated documentation merely because it exists.

### 1b. Map code ground truth

For each documentation area, identify the authoritative source for its claims.

Examples:

- Public API claims → actual exported functions, classes, types, methods, routes, or CLI definitions
- Function signatures → source declarations
- Key data shapes → types, schemas, serializers, or parsers
- How-to-run instructions → package scripts, `Makefile` targets, build configuration, or equivalent
- Dependency requirements → package manifests and lockfiles where appropriate
- Configuration options → schemas, parsers, defaults, and validation logic
- Architecture claims → directory structure, module boundaries, and imports
- Behaviour descriptions → the implementation and relevant tests
- Examples → the referenced API and, where possible, the project's actual toolchain

Explore according to relevance rather than an arbitrary file-count limit. Start with documentation and the code directly needed to verify its claims. Expand only when necessary.

If a claim cannot be confidently verified without substantially expanding the scope, report it as **unverified** rather than guessing.

### Phase 1 output

Produce a concise manifest:

| Doc file / location | Claims to verify |
|---|---|
| `README.md` → Installation | Package manager, runtime version, installation steps |
| `CLAUDE.md` → Commands | Build, test, and development commands |
| `src/auth/index.ts` → docstring | `login()` signature and behaviour |
| ... | ... |

If the requested scope is substantially larger than expected, ask whether the user wants to narrow it before continuing.

---

## Phase 2 — Audit

Work through every relevant item in the manifest. Verify claims against the current project state.

Do not rely on memory or assumptions when the claim can be checked directly.

### Claim types and how to verify them

| Claim type | How to verify |
|---|---|
| **Function/API signature** | Read the actual source; compare name, parameters, return type, and relevant semantics |
| **Behaviour description** | Trace the relevant code path and consult tests where available |
| **Example code** | Run, compile, or otherwise validate it when practical; otherwise manually verify against the current API |
| **How-to-run instruction** | Confirm the command exists and run it when safe and reasonably inexpensive |
| **Dependency/version requirement** | Check package manifests, lockfiles, toolchain configuration, or equivalent sources |
| **File path reference** | Confirm the path exists |
| **Cross-reference/link** | Confirm the target, heading, anchor, or URL resolves |
| **Architecture claim** | Verify against the current project structure and module relationships |
| **Configuration option** | Check the schema, parser, defaults, and validation logic |
| **CLI/API usage** | Verify against the actual command/API definition and, where practical, execute it |
| **Terminology/concepts** | Verify that terminology matches the project's current implementation and conventions |

Prefer automated verification where it is available.

For executable examples, commands, or other behaviour that can be validated through the project's existing test/build tooling, use that tooling rather than relying solely on manual reasoning.

### Findings classification

For every discrepancy, record:

- **Stale** — documents something that no longer exists or has materially changed
- **Missing** — an important, user-relevant behaviour, API, configuration option, workflow, or requirement exists but is absent where a reader would reasonably expect it
- **Inaccurate** — the documented thing still exists, but the description is incorrect
- **Broken example** — an example no longer works, compiles, or matches the current API
- **Broken reference** — a dead link, missing file, invalid anchor, or other unresolved reference
- **Unverified** — the claim could not be confidently verified within reasonable scope

Accurate, up-to-date claims require no finding.

Do not manufacture documentation problems simply because something is undocumented. Documentation should remain proportional to the project's needs and audience.

### Phase 2 output

Present a structured findings report before making edits:

```text
FINDINGS
========

README.md — Installation

  [Stale]          Node >=16 required; package.json says >=20
  [Missing]        Required GITHUB_TOKEN environment variable is not documented

CLAUDE.md — Commands

  [Inaccurate]     `npm run build` is now `npm run compile`

src/auth/index.ts — docstring

  [Stale]          login() no longer accepts a password parameter
  [Broken example] Example still calls login("user", "pass")