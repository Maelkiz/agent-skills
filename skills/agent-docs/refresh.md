# Mode: refresh

Detect documentation drift between `AGENTS.md` files and the actual codebase, then surgically fix stale content.

Invoked from `SKILL.md` Phase 1. All edits MUST conform to [guidelines.md](guidelines.md). On completion, return to `SKILL.md` Phase 2.

---

## R.1 — Detect drift

For each `AGENTS.md` in scope, check:

```bash
# Find all file paths referenced in the doc
grep -oP '`[^`]+\.[a-z]+`' <doc-file> | tr -d '`'
```

Verify each referenced path exists. Classify issues:

- **Critical** — referenced file doesn't exist, command doesn't work, or doc actively misleads (wrong directory, wrong flag)
- **Moderate** — line counts off by >30%, referenced config keys no longer exist, stale version numbers
- **Minor** — coverage gap: a package with ≥5 source files has no `AGENTS.md`

For each missing file, trace what happened:

```bash
git log --all --follow --oneline -- <missing-path> | head -5
```

- File moved/renamed → update reference to successor
- File deleted → remove reference; if the section is now vacuous, remove it

---

## R.2 — Fix critical issues

For each critical issue:

1. Read the affected doc section
2. Read the actual source to understand current state
3. Make a **targeted edit** — fix the specific issue, preserve surrounding context
4. If a file was deleted and the reference is irrelevant, remove it entirely

## R.3 — Fix moderate issues

For each moderate issue:

1. Read the affected doc section in context
2. Determine if drift is meaningful (does it actually mislead an agent?)
3. If yes: fix. If borderline: leave unless trivial.

**Line count updates:** `wc -l <file>`, round per [guidelines.md](guidelines.md) rounding rules, prefix with `~`.

## R.4 — Cover gaps (minor issues)

For packages with ≥5 source files and no `AGENTS.md`, generate a minimal doc:

```markdown
# AGENTS.md — <Package Name>

## Purpose

<1-2 sentences>

## Key Files

| File | Lines | Purpose |
|---|---|---|

## Test Command

\`\`\`bash
<exact invocation>
\`\`\`
```

Only generate for packages whose purpose would be non-obvious to an agent reading the code cold.

## R.5 — Severely drifted docs

If >40% of a doc's file references are broken, regenerate it using bootstrap's machinery ([bootstrap.md](bootstrap.md) §B.4) rather than patching:

1. Read all source files in the documented package
2. Regenerate using the appropriate template
3. Preserve any "Architecture" or design-decision sections that explain WHY — these age differently than factual references
4. Re-verify all paths and counts against the filesystem

---

**Done.** Return to `SKILL.md` Phase 2.
