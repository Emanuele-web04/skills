---
name: check-code
description: Review recent code changes across a repository to find critical or small bugs, integration risks, and feasible refactors. Use when a user asks to audit updates, assess how changes fit the whole repo, or decide whether refactoring is doable. Provide clear, non-technical summaries with actionable suggestions.
---

# Change Review & Refactor Scout

## Overview

Review code changes in context of the whole repo, identify likely bugs and integration risks, and propose feasible refactors. Communicate clearly and avoid overly technical wording while still being precise.

## Workflow

### 1. Establish the change set

- Check version control context (branch, status) and identify changed files.
- Use `git diff`, `git log -p`, and file history to understand what changed and why.
- Summarize the impacted areas (modules, APIs, config, tests, data).
- If scope is unclear (no VCS or too many changes), ask the user to narrow the range.

### 2. Map integration with the repo

- Trace call sites and dependencies with `rg`/search to see how changes connect to the rest of the codebase.
- Check configuration, routing, data contracts, and public APIs for compatibility.
- Note any mismatch between new behavior and existing expectations.

### 3. Scan for bugs and risks

- Look for missing checks, error handling gaps, off-by-one/edge cases, state inconsistencies, and unsafe assumptions.
- Categorize findings as `Critical` (likely to break or lose data) or `Small` (minor but real bug or sharp edge).
- If the change affects performance or security, call it out explicitly.

### 4. Assess refactor feasibility

- Suggest refactors only if they reduce complexity or risk without large behavior changes.
- State feasibility plainly (`Doable now`, `Doable with prep`, `Not worth it now`) and give a short reason.
- Estimate scope in simple terms (e.g., small/medium/large effort).

### 5. Respond in a clear, understandable format

Use this structure in the response:

- `Summary`: 2-4 sentences in plain language.
- `Findings (Critical)`: bullets with file references when possible.
- `Findings (Small)`: bullets with file references when possible.
- `Refactor Ideas`: bullets with feasibility and effort.
- `Questions / Assumptions`: bullets for anything that needs confirmation.
- `TLDR`: a very short closing recap in 1-3 bullets or 1-2 short sentences.

Keep wording simple. If a technical term is needed, add a short explanation in the same sentence.
For every complete review response, always include the `TLDR` section at the end, even when there are no findings.

## Output Quality Checks

- Ensure every finding ties to a specific change or integration point.
- Avoid speculative claims; label uncertainty clearly.
- Prefer actionable suggestions over vague advice.
