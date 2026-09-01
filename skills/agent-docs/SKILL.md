---
name: agent-docs
description: Create or update AGENTS.md documentation for a codebase — auto-detects whether to bootstrap from scratch or refresh existing docs
inputs:
  - name: scope
    required: false
    description: "Directory path relative to repo root (default: '.' for entire repo)"
  - name: mode
    required: false
    description: "'bootstrap' to generate from scratch, 'refresh' to fix drift, or omit to auto-detect"
---

# Agent Docs

Produces an accurate `AGENTS.md` hierarchy that an agent can trust cold. One workflow, two modes:

- **bootstrap** — no docs exist. Recon the codebase, mine gotchas, generate.
- **refresh** — docs exist but may have drifted. Detect → triage → surgical fixes.

All docs produced here MUST conform to **[guidelines.md](guidelines.md)** — read it before writing anything.

---

## Phase 0 — Resolve scope and mode

Use the user-provided scope, or default to `.`.

**Detect existing docs:**

```bash
find <scope> -name "AGENTS.md" | head -20
```

**Mode resolution:**

| User said | Docs found | Mode |
|---|---|---|
| nothing | none | `bootstrap` |
| nothing | exist | `refresh` |
| `bootstrap` | exist | Fill gaps only — bootstrap *uncovered* areas, refresh existing |
| `refresh` | none | Report "nothing to refresh", suggest bootstrap. Stop. |

---

## Phase 1 — Execute the mode

- **bootstrap** → follow **[bootstrap.md](bootstrap.md)**
- **refresh** → follow **[refresh.md](refresh.md)**

Both return here when done.

---

## Phase 2 — Shared tail

Every run finishes with these steps in order.

### 2.1 — Validate

Re-check that docs now match the code. For each doc this run created or edited:

```bash
# Verify every file path referenced in the doc actually exists
grep -oP '`[^`]+\.[a-z]+`' <doc-file> | tr -d '`' | while read f; do
  [ -e "$f" ] || echo "MISSING: $f"
done
```

Fix any broken references before continuing.

### 2.2 — Consistency sweep

Re-read every doc this run created or edited, plus its parent `AGENTS.md`. Check for contradictions: build commands that differ across docs, conventions stated differently at root vs nested, gotchas made obsolete by a fix. Contradicting instructions cause agents to pick one arbitrarily — resolve per the no-contradictions rule in [guidelines.md](guidelines.md).

### 2.3 — CLAUDE.md bridge

Claude Code reads `CLAUDE.md`, not `AGENTS.md`. Unless the user opts out or the repo already has a `CLAUDE.md`, create one at the repo root:

```markdown
@AGENTS.md
```

Add Claude Code-specific notes below the import only if needed. Keep it to the import line if there is nothing Claude-specific to add — one source of truth in `AGENTS.md`, shared across harnesses.

### 2.4 — Quality checklist

Before committing, verify:

- [ ] Every command in "Build & Test" works when copy-pasted
- [ ] Critical Gotchas are genuinely non-obvious (not "follow the style guide")
- [ ] No section is generic filler — every line earns its place
- [ ] File paths verified against the filesystem (`ls` / `find`)
- [ ] Line counts use actual `wc -l` values, rounded and prefixed with `~`
- [ ] Docs read as written by a knowledgeable engineer, not a template
- [ ] An agent reading cold could start productive work within 1 task

### 2.5 — Commit

Stage only the doc files this run touched:

```bash
git add <doc files changed>
git commit -m "docs: <bootstrap|refresh> agent docs — <summary>

Scope: <scope>
<bootstrap: docs created | refresh: Critical: X fixed, Moderate: Y fixed>
"
```
