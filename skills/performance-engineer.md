---
name: performance-engineer
description: Measure, diagnose, and safely improve software performance across critical paths such as startup, page load, API latency, database queries, frontend rendering, memory use, imports/bundle cost, batch jobs, file processing, and algorithmic hotspots. Use when the user asks to make code faster, reduce latency, profile bottlenecks, improve p95/p99, reduce memory/CPU, optimize startup time, speed up rendering, remove N+1 queries, or explain how to find performance wins without changing behavior.
---

# Performance Engineer

## Core Rule

Do not optimize vague code. Optimize a measured critical path.

Start by identifying what the user or system is waiting for, then capture a before number, attribute the cost to a cause, make the smallest useful change, and verify the after number and behavior.

## Default Workflow

1. Define the performance target:
   - Name the user-visible or system-visible path: startup, first useful render, search results, checkout, API response, import completion, background job throughput, memory peak.
   - Pick the metric that matters: wall time, p95/p99 latency, TTFB, TTI, TTFD, frame time, query count, CPU, memory/RSS, bundle size, allocations, throughput.
   - Separate cold, warm, dev, production, small input, large input, and realistic input.

2. Establish the baseline:
   - Find existing benchmarks, profiler traces, logs, debug flags, performance tests, or telemetry first.
   - If none exist, create a lightweight repeatable measurement before editing.
   - Run multiple samples when timing is noisy. Prefer median plus p95 or a min/median/max set over one lucky run.

3. Map the critical path:
   - Trace what happens before the target is reached.
   - Identify work on the hot path versus work that can happen later, once, in parallel, or not at all.
   - Inspect imports, initialization, render paths, request handlers, DB/API loops, cache boundaries, and large data transformations.

4. Classify the waste:
   - Too early: heavy work happens before it is needed.
   - Too often: repeated work could be reused, memoized, indexed, or batched.
   - Too much: more data, modules, DOM, records, or files are loaded than needed.
   - Too serial: independent work runs sequentially.
   - Wrong shape: the data structure or query shape forces repeated scans.
   - N+1: one item triggers one DB/API request each.
   - Render churn: UI recomputes or rerenders when state did not meaningfully change.
   - Cache hazard: cache is absent, overbroad, stale, or permission-unsafe.

5. Rank by impact and risk:
   - Prefer high-impact, low-risk changes first: lazy loading, batching, obvious indexing, removing duplicate work, parallel independent calls, reducing selected data.
   - Treat rewrites, cross-cutting caches, and semantic algorithm changes as higher risk and require stronger tests.
   - Do not patch every possible inefficiency. Scanner output and hunches are leads, not proof.

6. Patch conservatively:
   - Keep behavior, public APIs, output ordering, permissions, pagination, and error behavior unchanged unless the user explicitly asks otherwise.
   - Keep edits localized to the hot path and its helpers.
   - Add comments only where a performance choice would otherwise look accidental.

7. Verify:
   - Run focused tests first, then the broadest relevant build/type/lint/test command available.
   - Repeat the measurement under the same conditions.
   - Report before, after, improvement formula, tests, and residual risk.

## Senior Diagnostics

Before editing, write a short hypothesis in your own notes or user update:

```text
Hypothesis: <cost> is caused by <specific work> because <evidence>.
Verification: measure with <command/tool> and inspect <files/traces>.
```

Use this to avoid cosmetic performance patches.


## Common Moves

- Move heavy imports, initialization, config loading, network setup, or native bindings behind the branch or command that actually needs them.
- Replace repeated linear lookups with `Map`, `Set`, indexing, grouping, or precomputation when key equality is stable.
- Batch DB/API calls while preserving tenant filters, permissions, soft deletes, sorting, pagination, and error handling.
- Parallelize independent operations, but keep dependency order explicit.
- Load less: select fewer columns, paginate, stream, lazy-load, virtualize, or defer background work.
- Reduce render churn with stable props, memoized derived data, narrower state, virtualization, and moving expensive computation out of render.
- Add cache only when invalidation, user scope, permissions, and memory bounds are clear.

## Guardrails

- Never claim a performance win without before/after evidence.
- Do not trade correctness, security, accessibility, observability, or maintainability for a small speedup.
- Do not add broad caches around user-specific or permission-sensitive data unless the cache key and invalidation are proven.
- Do not deduplicate records only by display name or unstable labels.
- Do not change ordering, pagination, timezone handling, floating-point behavior, or mutation side effects casually.
- If measurement cannot be run, explain what blocked it and provide the exact command or method that should be used.

## Output Shape

For analysis-only requests, return findings ranked by likely impact:

- Target and baseline if available.
- Top bottlenecks with file/line references.
- Evidence for each bottleneck.
- Recommended fix.
- Expected impact and risk.
- How to verify.

For implementation requests, return:

- What changed.
- Before/after metrics if measured.
- Tests and checks run.
- Any residual risk or missing measurement.

## Detailed performance playbook

# Performance Playbook

## Measurement Choices

Pick a metric that matches the target.

| Target | Good metrics | Typical tools |
| --- | --- | --- |
| CLI startup | time to first output, warm/cold time, RSS | built-in debug command, `time`, hyperfine, process RSS |
| TUI/app startup | time to first draw, first useful render, import/init time | app debug flags, logs, browser/perf traces |
| Web page load | LCP, TTFB, INP, JS bundle size, hydration time | Lighthouse, WebPageTest, browser Performance panel |
| API endpoint | p50/p95/p99 latency, query count, payload size | logs, APM, curl loop, benchmark harness |
| Database path | query duration, count, rows scanned, plan | EXPLAIN, DB logs, ORM logging |
| Batch/import | throughput, peak memory, CPU, allocations | profiler, `/usr/bin/time -v`, heap snapshots |
| Frontend render | render count, frame time, expensive components | React profiler, browser Performance panel |

Prefer production-like builds and realistic data when the result is user-facing. Use local micro-benchmarks only to isolate a suspected cost.

## Waste Taxonomy

### Too Early

Heavy work happens before the selected path needs it.

Examples:
- CLI imports all command implementations to show help.
- Web app hydrates hidden panels before first useful content.
- Service opens external clients during process boot even for unrelated endpoints.

Senior moves:
- Dynamic imports or route-level lazy loading.
- Defer config, provider, native binding, or network setup to the narrowest handler.
- Split command declarations from command implementations.
- Move optional startup work to after first render or a background phase.

### Too Often

The same answer is recomputed.

Examples:
- Filtering/sorting large arrays during every render.
- Re-parsing config for every request.
- Rebuilding regexes, schemas, or lookup tables in loops.

Senior moves:
- Precompute once per request, render, process, or stable input.
- Memoize with explicit dependencies.
- Use maps/sets for stable-key lookup.
- Cache only with bounded scope and invalidation.

### Too Much

The code loads or processes more than needed.

Examples:
- Fetching full rows when only IDs and names are displayed.
- Loading every page of results before showing page one.
- Including admin-only UI in the initial bundle.

Senior moves:
- Select fewer fields.
- Paginate, stream, virtualize, or chunk work.
- Split bundles by route, command, or feature.
- Stop work after enough results are found.

### Too Serial

Independent operations wait for each other.

Examples:
- Three unrelated API calls run one after another.
- Startup waits for non-critical catalog loading.

Senior moves:
- Use `Promise.all` or framework equivalents for independent work.
- Start background work after the critical response or first draw.
- Preserve explicit ordering where side effects matter.

### Wrong Shape

The data model forces repeated scans or expensive joins.

Examples:
- Nested loops matching records by ID.
- `array.find` inside a loop over another large array.
- Sorting repeatedly instead of maintaining ordered data.

Senior moves:
- Index by ID with a map.
- Group once, then lookup.
- Use binary search, two-pointer scans, sweep-line algorithms, or pre-sorted data when the shape supports it.

### N+1

Each item causes another DB/API call.

Senior moves:
- Bulk fetch by IDs.
- Join/preload/aggregate in the database.
- Batch API calls.
- Preserve auth, tenant, soft-delete, filtering, ordering, and pagination constraints.

### Render Churn

UI work repeats without user-visible need.

Senior moves:
- Move expensive derived data out of render or memoize it.
- Keep props stable.
- Narrow state ownership.
- Virtualize long lists.
- Avoid layout thrash: batch DOM reads/writes and avoid sync measurement loops.

## Impact/Risk Ranking

Prioritize like this:

| Priority | Type | Examples |
| --- | --- | --- |
| First | High impact, low risk | lazy import unused path, batch obvious N+1, precompute stable lookup |
| Second | High impact, medium risk | query rewrite, render ownership change, endpoint-level cache |
| Third | Low impact, low risk | remove duplicate parse, cheap memo, minor allocation cut |
| Avoid unless proven | Low impact, high risk | broad rewrite, clever algorithm with unclear behavior, global cache |

## Evidence Standards

Good evidence:
- Before/after timings from the same command and environment.
- Profiler trace showing cost concentration.
- Query count reduction.
- Bundle size/import graph difference.
- Memory/RSS reduction.
- Tests proving behavior unchanged.

Weak evidence:
- "This looks cleaner."
- One unrepeatable timing.
- A micro-benchmark for a path that is not hot.
- Scanner output without confirming runtime relevance.

## Before Editing Checklist

- What exact path is slow?
- What is the before metric?
- Is this path hot or user-visible?
- What is the suspected cause?
- What is the smallest change that could remove or defer the cost?
- What behavior must remain unchanged?
- Which tests or smoke checks prove that?

## After Editing Checklist

- Did the same measurement improve?
- Were tests run?
- Did memory, CPU, query count, or bundle size change as expected?
- Did the patch preserve ordering, permissions, pagination, and errors?
- Is any cache bounded and correctly invalidated?
- Is the final explanation honest about missing measurements?

## Evidence templates

# Evidence Templates

## Analysis Report

```markdown
## Performance Target

- Path:
- Metric:
- Environment:
- Baseline:

## Findings

### 1. <Finding title>

- Location:
- Category: Too early / Too often / Too much / Too serial / Wrong shape / N+1 / Render churn / Cache hazard
- Evidence:
- Current behavior:
- Recommended change:
- Expected impact:
- Risk:
- Verification:

## Suggested Order

1. <Highest impact, lowest risk>
2. <Next best change>
3. <Deferred because risk/uncertainty>
```

## Implementation Summary

```markdown
## What Changed

- <Concrete change>

## Evidence

- Before:
- After:
- Improvement:
- Measurement command:

## Verification

- <Test/check command>: passed/failed/not run

## Residual Risk

- <Only include real remaining uncertainty>
```

## Improvement Formula

For lower-is-better metrics such as latency:

```text
improvement percent = ((before - after) / before) * 100
```

For higher-is-better metrics such as throughput:

```text
improvement percent = ((after - before) / before) * 100
```

Always include units and avoid comparing cold numbers to warm numbers unless the distinction is explicit.
