# Mode: bootstrap

Generate agent documentation from scratch for a codebase with no existing docs.

Invoked from `SKILL.md` Phase 1. All output MUST conform to [guidelines.md](guidelines.md). On completion, return to `SKILL.md` Phase 2.

---

## B.1 — Repo reconnaissance

Systematically characterize the codebase before writing anything. Gather facts; don't interpret yet.

**Build system** — identify by indicator file:

| Indicator | Build System |
|---|---|
| `package.json` (root) | npm/yarn/pnpm — check for workspaces |
| `turbo.json` | Turborepo monorepo |
| `Cargo.toml` | Cargo (Rust) — check for workspace members |
| `go.mod` | Go modules — check for multi-module layout |
| `Makefile` | Make — read targets |
| `Gemfile` | Bundler (Ruby) — check for `bin/rails` |
| `pom.xml` / `build.gradle` | Maven/Gradle |
| `BUILD` / `WORKSPACE` | Bazel — BUILD files may be auto-generated |

Record: build command, test command (all tests + single file/package), lint/format commands, any codegen steps.

**Languages & frameworks** — version constraints, framework version, package manager.

**Entry points** — main server/CLI entry, route/controller definitions, config loading.

**Module inventory** — list top-level modules. For each: name, approximate file count, one-sentence responsibility.

**Test infrastructure** — test framework, test location convention (`__tests__/` vs mirrored), how to run a single test.

---

## B.2 — Gotcha discovery

Highest-ROI step. Mine everything that will silently break an agent.

**Linter & formatter configs** — read `.eslintrc*`, `tsconfig.json`, `.rubocop.yml`, `.prettierrc*`, `.editorconfig`, etc. Extract: non-obvious rules that would trip up an agent (fatal warnings, import ordering, naming conventions enforced by tooling). Skip obvious ones like semicolons.

**CI pipeline** — read `.github/workflows/*.yml`, `.circleci/config.yml`, `Jenkinsfile`, `.pre-commit-config.yaml`. Extract: mandatory checks that gate a merge, auto-formatting steps, deployment triggers.

**Generated/protected files** — identify files agents must NEVER edit. Look for "DO NOT EDIT" headers, `generate-*` scripts, Makefile `generate` targets.

**Git history archaeology:**

```bash
# Recurring fix patterns = gotchas worth documenting
git log --oneline --since="3 months ago" --grep="fix" -- <scope> | head -20

# Most-changed files = complexity hotspots
git log --since="3 months ago" --name-only --pretty=format: -- <scope> | sort | uniq -c | sort -rn | head -15
```

**Implicit conventions** — read 5-10 representative files across different areas. Look for: file naming patterns, directory organization, naming suffixes (`*Service`, `*Handler`), error handling patterns, logging patterns.

---

## B.3 — Hierarchy design

```
Is it a monorepo with >3 distinct packages/services?
├── YES → hierarchical: root AGENTS.md + one AGENTS.md per major module
└── NO  → flat: single AGENTS.md at repo root
    └── Library with >30 source files? Consider hierarchical.
```

A single accurate root AGENTS.md is an acceptable first bootstrap pass even for a hierarchical repo, if it stays within ~250 lines. Split into per-module docs as a follow-up once the root outgrows that budget. Prefer shipping one accurate doc over blocking on a full hierarchy.

---

## B.4 — Generate documentation

Using facts from B.1–B.2, write docs following [guidelines.md](guidelines.md).

**Root AGENTS.md template:**

```markdown
# AGENTS.md — <Repo/Project Name>

## Purpose

<1-3 sentences: what this codebase does and who uses it>

## Architecture

<Only if multi-component. Brief topology: services, how they connect>

## Module Layout

<Table or bullet list of top-level modules with one-line descriptions>

## Key Files

| File | Lines | Purpose |
|---|---|---|

## Build & Test

<Exact copy-pasteable commands: build, test (all + single), lint, format>

## Code Conventions

<Rules from B.2 linter analysis and implicit convention discovery. CORRECT/WRONG examples.>

## Critical Gotchas

<Numbered, most dangerous first. Bold one-line summary + why it's dangerous.>

## Terminology

<Domain terms agents won't know from general training>

## Do

<One-line actionable items>

## Don't

<One-line prohibitions — especially non-obvious ones>
```

**Subproject AGENTS.md** (hierarchical only) — for each significant subproject:
- Purpose (1-2 sentences in context of parent)
- Key Files table
- Build & Test (specific targets)
- Code Conventions (only subproject-specific rules)
- Critical Gotchas (subproject-specific)

Target: ~80–150 lines each.

---

**Done.** Return to `SKILL.md` Phase 2.
