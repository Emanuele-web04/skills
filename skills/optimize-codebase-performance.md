---
name: optimize-codebase-performance
description: "Run a bounded, evidence-driven performance optimization workflow across a codebase and its surrounding infrastructure. Use for requests to reduce excessive CPU, RAM, GPU, latency, I/O, or rendering costs; profile and optimize application, dependency, infrastructure, and UI layers; and produce before/after benchmarks with percentage changes."
---

# Optimize Codebase Performance

Treat performance work as a persistent engineering loop: establish a trustworthy baseline, find measured bottlenecks, make small safe changes, re-measure the same workload, and stop when the evidence says the target is met or further work is no longer justified.

## Operating contract

- Act as a Senior/Staff Performance Engineer. Own the investigation through implementation, validation, and the final benchmark report.
- Use the user's requested target explicitly: CPU, RAM, GPU, latency, I/O, rendering, cost, or a combination. Do not optimize an unmeasured proxy.
- Keep the loop bounded by a TODO/goal plan, available budget, correctness, and explicit stop conditions. Never run an unbounded or literal infinite loop.
- Do not run Git commands during the optimization run. Use the filesystem, profilers, tests, and benchmark artifacts as the source of truth.
- Treat subagents as auditors: they may inspect code, profiles, and benchmark results and return findings, but they must not modify files.
- Use external research only for non-obvious opportunities. Prefer version-matched official documentation, Context7 when available, and targeted web search; record the source and verify that the recommendation applies before changing code.
- Keep implementation local and reversible where possible. Preserve behavior and add or update tests when the optimization changes semantics.

For requests to inspect, diagnose, or plan, report findings without editing. For requests to optimize, make the requested in-scope local changes and run relevant non-destructive validation. Ask before production changes, destructive actions, purchases, permission changes, external writes, or sending sensitive project data to third-party services.

## Workflow

### 1. Scope and baseline

Create a TODO/goal plan with these checkpoints:

1. Scope, environment, target metrics, workload, and success criteria.
2. Baseline measurements and reproducible commands.
3. Bottleneck analysis by layer.
4. Candidate changes, risk, and expected metric impact.
5. Implementation and correctness checks.
6. Independent audit and re-measurement.
7. Final benchmark and handoff.

Before major tool calls, give a short preamble stating the decision and reason, for example: “I’m profiling the representative workload first so optimization choices are tied to measured peaks.” Keep preambles for notable steps, not every trivial command.

Inspect the codebase and its surrounding runtime/deployment configuration before editing. Record the relevant OS, hardware, runtime, framework, dependency, build, container, and service settings. Establish a representative workload and capture, where applicable:

- CPU: wall time, average and peak utilization, hot functions, thread or worker behavior.
- RAM: resident/set size, peak allocation, allocation rate, retained objects, cache growth.
- GPU: utilization, memory, kernel or shader hotspots, transfer/copy overhead.
- Service behavior: latency percentiles, throughput, queue depth, I/O, errors, and cost.
- UI: render time, layout/paint time, frame rate, component render frequency, network and image costs.

Save enough raw evidence to reproduce the baseline. Never claim a percentage improvement without paired baseline and post-change measurements from the same workload and comparable conditions.

### 2. Analyze all relevant layers

Choose the layers that exist in the project; do not force irrelevant analysis.

- Application: algorithms, repeated work, allocations, copies, synchronization, concurrency, batching, serialization, logging, resource lifetime, and error paths.
- Dependencies and libraries: version-specific hot paths, defaults, transitive cost, build flags, native bindings, and safer lower-cost alternatives.
- Infrastructure: worker counts, connection/client pools, queues, caches, container limits, autoscaling, image/build settings, storage, network, observability, and GPU/runtime configuration.
- UI and design components: inspect each meaningful component for unnecessary renders, expensive effects, layout thrash, large lists without virtualization, image decoding, animation work, and redundant requests. Use a browser or simulator only when it is available and the task is in scope.
- Data and workload shape: identify whether peaks come from input size, concurrency, cold starts, cache state, or a specific request path.

For each suspected hotspot, state the evidence, mechanism, proposed change, expected trade-off, and measurement that can falsify the hypothesis. Prefer the smallest change that tests the hypothesis.

### 3. Implement in measured batches

Group only closely related changes. For every batch:

1. State the hypothesis and expected metric movement.
2. Make the smallest in-scope local edit.
3. Run focused correctness checks, then the representative benchmark.
4. Compare against the baseline and the immediately prior version.
5. Revert or revise changes that fail correctness, worsen the target materially, or introduce an unjustified trade-off.
6. Update the TODO/goal with evidence and the next decision.

Do not cargo-cult advice from research or auditor reports. Do not optimize code that is not on the measured path unless the change is a clearly justified preventive fix.

### 4. Audit and research

After each meaningful batch, run an independent audit if subagents are available. Give auditors the relevant files, profiles, and benchmark output; ask them to look for missed hotspots, regressions, measurement flaws, and unsafe assumptions. Do not give them permission to edit.

Use targeted external research when local evidence leaves a gap, such as a framework default, allocator behavior, GPU/runtime setting, or deployment limit. Record the URL, applicable version, recommendation, and why it fits this workload. Keep research read-only and do not expose private code or credentials.

If an optional second-opinion tool such as `claudex` is available and the user has authorized its use, use it only as an additional auditor. Treat its output as hypotheses that still require local measurement.

### 5. Stop deliberately

Stop the loop when any of the following is true:

- The requested target and correctness criteria are met and the benchmark is reproducible.
- Further iterations show diminishing or inconsistent returns.
- The remaining bottleneck requires a material architecture, infrastructure, or product decision outside scope.
- The change would increase risk, cost, latency, or complexity more than the measured gain justifies.
- The available time, compute, or tool budget is exhausted.

At stop time, mark the goal achieved only when the success criteria are verified. Otherwise report the remaining blocker, evidence, and the smallest next experiment.

## Benchmark and handoff


- workload, environment, commands, sample count, warm-up policy, and variance;
- baseline and optimized values for every target metric;
- absolute delta and percentage reduction/increase;
- the function, library, component, or infrastructure setting responsible for each material change;
- correctness/test status, regressions, trade-offs, limitations, and recommended next steps.

Use this formula for metrics where lower is better:

```text
percentage reduction = (baseline - optimized) / baseline * 100
```

Report throughput increases separately rather than calling them reductions. Preserve negative results; they prevent repeated work and make the optimization decision auditable.

## Prompting and response discipline

- Keep instructions lean: state each rule once, expose only relevant tools, and retain examples only when they encode a real requirement or fix a measured reliability gap.
- Define the role, target, constraints, evidence requirements, success criteria, and output format up front.
- Lead the final response with the result. Preserve decisions, evidence, material caveats, and next actions; omit repetition and generic reassurance.
- Use a table for repeated benchmark fields and concise prose for interpretation. Use the project's established formatting conventions when they exist.

## Benchmark rubric

# Benchmark rubric

Use this reference when the optimization run reaches validation or handoff. Keep the protocol short enough to repeat and strict enough to make comparisons meaningful.

## Measurement protocol

Record:

- target metric and whether lower or higher is better;
- exact workload, input shape, concurrency, and cache state;
- hardware, OS, runtime, framework, dependency, build, and deployment settings that affect results;
- command or script used, warm-up policy, sample count, and whether samples were sequential or concurrent;
- mean/median and p95/p99 or peak values where meaningful, plus variance or confidence interval;
- unrelated load, throttling, background services, and any known confounders.

Run baseline and optimized measurements under the same protocol. Repeat noisy workloads until the direction and magnitude of the result are stable enough to support the decision. If conditions changed, label the comparison as directional rather than causal.

## Report table

Use one row per material optimization:

| Layer | Hotspot / change | Workload | Baseline | Optimized | Absolute delta | % reduction / increase | Evidence | Trade-off |
|---|---|---|---:|---:|---:|---:|---|---|
| Application / dependency / infrastructure / UI | Function, library, component, or setting | Reproducible case | Value + unit | Value + unit | Optimized - baseline | Formula below | Profiler, trace, test, or benchmark | Memory, latency, cost, complexity, or none |

For a lower-is-better metric:

```text
reduction % = (baseline - optimized) / baseline * 100
```

For a higher-is-better metric, report the improvement separately:

```text
increase % = (optimized - baseline) / baseline * 100
```

## Acceptance checklist

- [ ] Every claimed change has paired measurements from the same workload.
- [ ] The metric direction and units are explicit.
- [ ] Correctness tests and behavior checks pass.
- [ ] Peak and tail behavior were checked when the target is resource stability or latency.
- [ ] Regressions and negative results are included.
- [ ] External recommendations are linked and version-matched.
- [ ] Remaining bottlenecks and the next smallest experiment are stated.
