# Agents Skills

## What is this?

This is a set of reusable agent skills designed to improve context management, agentic coding performance, and enforce good practices, all while keeping you in the loop to ensure alignment.

## The Skills

### `/phased-plan`
Breaks any multi-file feature or refactor into commit-sized phases — each with a clear goal, concrete file changes, success criteria, and quality gates. Claude pauses after every phase for your review before committing, so you stay in control of large changes while ending up with a clean, reviewable git history.

### `/prune`
Surgically trims a bloated session without losing the context that matters. It collapses absorbed tool outputs, completed sub-tasks, and redundant context into one-liners — while explicitly protecting the things `/compact` routinely loses: *why* a decision was made, rejected alternatives and their reasons, and active constraints. Run it mid-task when the session feels heavy, or right before `/compact` to get a sharper summary.

### `/prunepact`
One command that runs `/prune` then `/compact` back-to-back. Prune removes the dead weight first so the compaction summary is built from clean signal rather than noise. The result is a smaller, more accurate context than normal `/compact` and greater context window reduction than just `/prune`. This skill results in the greatest context window reduction overall, but is more lossy than `/prune`.