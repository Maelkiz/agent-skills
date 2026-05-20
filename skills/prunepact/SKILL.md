---
name: prunepact
description: >
  Runs /prune followed by /compact in sequence for maximum context reduction
  while preserving what matters. Use when a session is getting too bloated
  and you want the deepest possible cleanup without losing decisions, rejected
  alternatives, or active constraints. /prune runs first to surface and protect
  high-value signals; /compact then summarises the cleaned context so the
  resulting summary is far more accurate than a raw compaction would produce.
disable-model-invocation: true
---

# /prunepact — Prune then Compact

Run these two skills in order:

1. `/prune $ARGUMENTS` — removes dead weight and produces the pruned context
   block with decisions, constraints, and open work preserved
2. `/compact` — summarises the now-clean context into a compact summary

The result is a smaller, sharper context than either command achieves alone.