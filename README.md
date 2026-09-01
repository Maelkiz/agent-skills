# Agents Skills

## What is this?

This is a set of language agnostic agent skills for improving context management ergonomics and enforcing good practices while keeping you in the loop to ensure alignment.

## The Skills

### `/phased-plan`
Breaks any multi-file feature or refactor into commit-sized phases — each with a clear goal, concrete file changes, success criteria, and quality gates. Claude pauses after every phase for your review before committing, so you stay in control of large changes while ending up with a clean, reviewable git history.

### `/handoff`
Compacts the current conversation into a structured briefing for a fresh agent to pick up. Use this instead of `/compact` when you want a clean slate — the new session inherits only what matters, rather than a compressed version of everything.

### `/update-docs`
Audits and syncs documentation against the actual code. Works across READMEs, `docs/`, inline docstrings, and agent context files (`AGENTS.md`, `CLAUDE.md`). Detects drift via broken references and git age gaps, writes all findings before touching any files, then fixes stale content, removes cruft, and fills gaps.

### `/agent-docs`
Creates or updates an `AGENTS.md` hierarchy for a codebase — auto-detects whether to bootstrap from scratch or refresh existing docs. Mines the repo for gotchas, conventions, and build/test commands that would otherwise trip up an agent working cold.

### `/write-tests`
Writes a thorough test suite for a given piece of code. Discovers the project's existing test framework and conventions, then builds a matrix covering normal behaviour, boundary conditions, error cases, and state transitions — verifying through public interfaces rather than implementation details.
