---
name: refactor-code
description: Refactor existing code into a modular, ordered structure without breaking behavior. Use when users ask to clean up architecture, split large files/functions, reduce complexity, and improve maintainability while preserving current functionality.
---

# Refactor Code

## Overview

Turn messy or growing code into a clean, modular, and predictable structure.
Keep behavior stable unless the user explicitly asks for behavior changes.
Prioritize readability, clear boundaries, and future-safe organization.

## Workflow

### 1. Confirm refactor intent and boundaries

- Identify the refactor target (files, modules, feature area).
- Define non-negotiables: behavior parity, API compatibility, performance constraints.
- Clarify what is in scope vs out of scope before changing structure.

### 2. Map current structure

- Inspect current entrypoints, call flow, and dependencies.
- Detect pain points: oversized files, mixed concerns, duplicated logic, unclear naming.
- Note hidden coupling (shared state, side effects, implicit contracts).

### 3. Design target modular structure

- Propose a small set of focused modules with single responsibilities.
- Define each module's public interface (inputs, outputs, errors).
- Separate concerns clearly (orchestration, domain logic, parsing/validation, I/O, UI rendering).
- Keep naming consistent and predictable across modules.

### 4. Refactor in safe slices

- Apply changes incrementally in logical steps, not one giant rewrite.
- Keep each step buildable and testable.
- Add adapter/shim code temporarily when needed to avoid breaking callers.
- Remove dead code only after confirming replacements are wired and covered.

### 5. Preserve behavior with verification

- Run targeted tests after each major slice.
- Add or update regression tests for edge cases touched by refactor.
- Verify key contracts (API payloads, schema fields, UI output, error handling).
- Call out any intentional behavior changes explicitly.

### 6. Finish with cleanup and documentation

- Simplify naming and folder structure where helpful.
- Ensure code paths are easy to follow top-to-bottom.
- Add brief comments only where logic is non-obvious.
- Document remaining technical debt and future refactor opportunities.

## Refactor Guardrails

- Do not mix feature changes with refactor unless asked.
- Avoid unnecessary abstraction layers.
- Prefer pure functions for reusable logic.
- Keep module boundaries explicit and easy to test.
- Optimize for maintainability over cleverness.

## Response Format

Use this structure in your response:

- `Refactor Goal`: what was cleaned up and why.
- `What Changed`: concise list of structural changes.
- `Behavior Safety`: how parity was preserved.
- `Validation`: tests/checks run and results.
- `Future Improvements`: optional, practical next steps.

## Output Quality Checks

- Every structural change has a clear purpose.
- Refactor reduces complexity rather than moving it around.
- Integration points remain compatible or are explicitly migrated.
- Tests cover the highest-risk refactor points.
