# KEA agent flow

## Start from the best available evidence

Identify the objective, workload, correctness and resource constraints, and what the clock
includes. Reuse project harnesses and evidence that match the code, binary and workload.

With a trusted baseline, a usable correctness check and a concrete idea, start with a
[focused paired screen](measurement.md#screen-the-mechanism-on-informative-cases). If the next
decision depends on a missing baseline, uncertain dispatch or an unclear bottleneck, fill that gap
first. A short note in the experiment log is enough for a small experiment. Use a fuller plan
when dependencies or risks make it useful.

Before GPU execution, apply project rules, verify any applicable hard limits and assess memory
and execution risks for the cases being run. Use the
[memory-risk guidance](measurement.md#before-candidate-execution) when peak growth or device
headroom is uncertain. Check synchronization and replay compatibility when profiling stateful
or persistent kernels.

## Search and learn

Use source, timings, analytical bounds, profiles or references as they help answer the current
question. [Architecture search](architecture-search.md) covers diagnosing the whole path,
including dispatch, dependencies, overlap and memory lifetimes. For hardware-specific questions,
KernelWiki and an appropriate profiler are available when useful.

Choose experiments for expected payoff and what they can teach. Try local changes, architectural
alternatives or coupled changes when their dependencies justify testing them together. Run
independent experiments in parallel when authorized and when their measurements remain comparable.
Scale validation to the experiment. Use focused tests for exploration and the actual workload for
results claimed against the task objective.

Keep the baseline and useful candidates recoverable. Use the project's records or the
[experiment log](experiment-records.md) to capture enough evidence to reproduce consequential
results and explain decisions.

When an experiment gives no usable gain, use the
[no-gain guidance](experiment-records.md#interpret-a-no-gain-result) to distinguish what happened
to the candidate from what the evidence says about the direction. Choose a follow-up when it could
change the decision. When the search stalls or findings change the bottleneck, reconsider the
[remaining opportunities](architecture-search.md#choose-the-next-experiment).

A substantial speedup above the memory target can remain the starting point for performance work
within hard limits and device headroom. Keep the best in-target alternative, then
[recover memory or request review](measurement.md#recover-memory-on-a-promising-candidate)
if the final result needs a tradeoff.

## Validate the result

Verify that the integrated call path runs the intended implementation. Apply the task's
[correctness, memory and timing criteria](measurement.md) on the required workloads. Use
comparable, uninstrumented measurements for claimed speedups and check relevant regressions.
Tie the evidence to the delivered code and configuration.

Report results, remaining tradeoffs and any unmet criteria. If no candidate qualifies, preserve
the best verified result and explain what blocked delivery or still needs a decision.
