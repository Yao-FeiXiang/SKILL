---
name: simplify
description: Use when reducing complexity in code, architecture, documentation, or implementation plans while preserving behavior, interfaces, and future extensibility.
---

# Simplify

## Overview

Reduce moving parts without reducing correctness. Prefer fewer concepts, fewer transitions, fewer states, and fewer special cases.

## When to Use

- The current design has too many layers for the delivered behavior.
- Several abstractions exist but only one is actually used.
- The code works, but the control flow is hard to follow.
- A document or interface is correct but bloated.
- You need to keep extension points, but the current structure is overbuilt.

## Core Rule

Simplify structure first, not semantics.

Keep:

- external behavior
- public interface contracts
- required extension points
- verification paths

Remove or collapse:

- speculative abstraction
- duplicate representations
- unused indirection
- split logic that always changes together
- configuration surface with no meaningful choice behind it

## Process

1. Identify the invariant behavior that must not change.
2. Identify which layers are essential and which are speculative.
3. Collapse representations that express the same fact twice.
4. Merge units that always move together.
5. Keep one obvious place for each decision.
6. Re-run verification after each simplification.

## Heuristics

- If an abstraction has one implementation and no realistic second one soon, inline or collapse it.
- If a data structure exists only to be converted immediately into another, remove one of them.
- If a pipeline stage has no independent invariants, merge it into the adjacent stage.
- If a feature will evolve later, keep the seam but simplify the implementation behind it.

## For Compilers

- Preserve the phase boundaries that affect correctness: parsing, semantics, IR, lowering, emission.
- Simplify inside a phase before deleting a phase boundary.
- Keep optimization behind explicit switches.
- Keep language-variation hooks explicit rather than scattering conditionals.

## Common Mistakes

- Deleting extension points that the project actually needs.
- Replacing a clear layered design with tangled direct calls.
- Simplifying by hiding behavior instead of removing complexity.
- Dropping verification because the new version “looks smaller”.
