# Measurement and validation

Use this reference for the contract, baseline, candidate comparison and acceptance. Prefer the
project's existing harness and acceptance policy; record deviations and measurement blind spots.

A measurement excludes preparation from latency only when the task's actual timing contract does so;
the resources held by preparation may still count toward peak memory.

## Freeze the contract

- Identify hardware and device count, code/build identity, dtype, shapes, strides/layouts, input
  distribution, seeds, concurrency and execution mode (including graph capture if applicable).
  Select representative and boundary cases; keep a holdout for shape generalization when in scope.
  State any weighting and per-case regression limits. Follow the holdout procedure below.
- Define exactly which dispatch, allocations, preparation, synchronization, forward, saved state and
  backward are inside the timing and memory boundaries. Record warmup, repetitions, estimator, noise
  treatment, reset requirements and exact commands. Record user-specified time, GPU, iteration or
  cost limits and stop conditions when provided. Preserve the frozen acceptance inputs, reference
  and thresholds; a changed contract requires a new comparison.
- Verify the oracle rejects a controlled incorrect output for affected behavior. Restore temporary
  changes and run the correct baseline when one exists. An invalid or unavailable oracle blocks
  claims of correctness; a single negative control proves only that particular error is detected.

### Holdout and evaluator boundary

When shape generalization is in scope, define the valid domain and search/acceptance split before
tuning. Record separately whether holdout feedback influenced tuning and whether its cases, inputs
and results were hidden from the optimizing agent; a request not to inspect them is not isolation.

When the contract requires baseline comparison, obtain baseline performance and the applicable
per-case memory ceiling for holdout cases.

If an independent evaluator exists, let it retain those details and apply the appropriate per-case
memory protection to each holdout execution without exposing protected information to the search.
Otherwise record what was exposed; do not claim the holdout was hidden.

Freeze the candidate identity before final holdout evaluation and record feedback released to the
optimizer. If any holdout feedback influences tuning, mark those results exploratory and use fresh
cases for an independent generalization claim, or explicitly report that no independent holdout is
available.

If the contract requires independent holdout evidence and none is available, treat that acceptance
criterion as unresolved.

Prefer the project's trusted evaluator and immutable input/contract snapshots, when available.
Before accepting results, verify that the actual evaluator, oracle, inputs, thresholds and candidate
source/build identities match the frozen contract and that the full acceptance matrix ran.

Preserve original evaluator outputs; a candidate's own summary is not a substitute. If tests or the
evaluator must change, record why and re-establish the affected baseline and evidence; honor
user-authorized test changes.

Where the same agent can alter both the evaluator and records without enforcement, describe this as
a procedural check, not tool-enforced integrity. Do not create a new isolation service solely to
satisfy this step.

## GPU memory

Choose a metric matching the user's limit and state its device/process scope, units and coverage.
Measure the full relevant lifetime, especially from forward through backward, and keep allocations
that are live at the boundary. Include model inputs, outputs, saved tensors, persistent workspace
and temporary peaks if they fall within the contract; do not sum independent component peaks as if
they occurred together.

- Framework **allocated** peak describes live allocations tracked by the allocator; framework
  **reserved** peak includes its cached blocks. Neither automatically includes allocations from
  other CUDA libraries, custom allocators or other processes. Report both when useful, but compare
  the contract's selected metric against the same metric and scope.
- For a process/device-wide limit, use an available method that covers the relevant allocations and
  lifecycle. Record what the tool sees, whether it samples or tracks allocations, sampling
  resolution, other processes and any graph-pool or external-allocation blind spots. A sampled
  maximum may miss a short-lived peak; device-wide usage with other processes is not a per-process
  measurement.
- Warm up and synchronize as needed, reset peak statistics without freeing state that belongs in the
  measured lifetime, execute the complete path, synchronize, then read peaks. Use comparable
  allocation and execution conditions for baseline and candidate when both exist, including
  persistent and graph pools. Record absolute peak for each case and device, baseline delta when
  available, plus workspace and saved-state changes when observable.
- If no explicit budget exists, the default ceiling is the corresponding measured baseline peak per
  workload and selected scope. A candidate with a suspected excess needs a revised authorized budget
  before execution or promotion. Estimates are not measurements. If the needed measurement coverage
  or a baseline required by the contract is unavailable, document the gap, continue safe
  investigation, and do not report the memory check as passed.

### Discover and verify budget protection

Before GPU execution, identify the execution host or scheduler, driver and runtime versions, GPU
UUIDs, visible devices for each rank, relevant allocator and allocation types, process tree and
existing authorized limits.

Inspect the target environment itself: local, container and remote GPU capabilities may differ. Use
the existing platform's read-only status interfaces first; do not change shared GPU or cgroup
configuration just to discover capabilities.

On NVIDIA/Linux, an initial read-only check can use `nvidia-smi -L`,
`nvidia-smi --query-gpu=name,driver_version,memory.total,mig.mode.current --format=csv,noheader`, `uname -r`,
`stat -fc %T /sys/fs/cgroup` and `cat /proc/self/cgroup`. These identify available devices and
cgroup layout, not an effective GPU memory limit. Query the actual client/cgroup using the
platform's supported status interface; correlate its devices by UUID, including each rank and child
process.

For each applicable limit, map its accounting scope to the contract's selected peak metric and
lifetime. Check the configured and effective limit, process/cgroup or client membership, each device
and child process, and coverage of framework, library, custom and managed allocations. Record what
is verified and any blind spots.

A limit's numeric value need not mean the same thing as a framework allocated/reserved peak:
context, caches and accounting scope can differ.

If the mechanism changes GPU partitioning, visible SMs, memory or execution conditions, obtain a
comparable baseline under those conditions; do not present that timing as a result for a different
production configuration without separately verifying it.

To verify enforcement, use a platform-provided status check or an isolated, controlled probe only
when it can be run within the authorized resource boundary. Do not fill a shared GPU or assume an
environment variable took effect.

If effective scope or enforcement cannot be verified, mark protection unavailable and use the
blocked path below.

An explicit user or project limit applies to baseline, warmup, negative-control and candidate GPU
executions. When the default ceiling is derived from the first baseline, that ceiling does not yet
exist during its measurement: respect the environment's existing resource boundaries, document the
gap, and do not claim that the not-yet-established baseline ceiling was enforced.

Once established, protect all subsequent candidate GPU execution, including warmup, smoke,
correctness and profiling, over the contract's full lifetime on every device:

- Use a verified runtime limit or isolation mechanism that prevents allocations beyond the
  applicable ceiling, including relevant framework, library and custom allocations; record its
  configured limit and coverage. A framework allocator cap covers only allocations controlled by
  that allocator unless additional protection covers the rest.
- Otherwise use a conservative, justified upper bound on the simultaneous live allocations for that
  exact workload and execution path, including external allocations and fixed overhead; establish
  that the bound does not exceed the ceiling. A rough estimate alone is not such a bound.

Peak measurement and periodic monitoring can detect excess after it occurs; they are not hard
prevention. If neither protection covers the required scope, leave candidate GPU execution blocked,
record the gap and continue safe inspection or planning.

Do not claim the budget was enforced. A user-authorized budget or scope change requires a new
contract; it does not turn incomplete protection into enforcement. After execution, still measure
the peak under the contract for acceptance.

PyTorch's [per-process
fraction](https://docs.pytorch.org/docs/stable/generated/torch.cuda.memory.set_per_process_memory_fraction.html)
limits its caching allocator. Check the effective coverage of any available mechanism rather than
assuming it applies. For metric semantics see
PyTorch's [allocated
peak](https://docs.pytorch.org/docs/stable/generated/torch.cuda.memory.max_memory_allocated.html),
[reserved
peak](https://docs.pytorch.org/docs/stable/generated/torch.cuda.memory.max_memory_reserved.html) and
[CUDA memory management](https://docs.pytorch.org/docs/stable/notes/cuda.html#memory-management).

## Compare performance and correctness

Measure baseline and candidate under the same stated conditions, separately from
profiler-instrumented runs. Before deciding whether a difference is meaningful, establish per-case
measurement variability using repeated comparable samples or an applicable validated project noise
model; record its conditions and estimator. If variability cannot be established, report an apparent
gain as uncertain rather than a demonstrated improvement. If there is no baseline and the contract
specifies an absolute target, measure the candidate against that target without claiming a
comparative speedup.

Keep raw samples and per-case results. If the improvement is comparable to noise, interleave
baseline/candidate runs (e.g. ABBA) when practical, repeat enough to characterize variability, and
report the estimator and uncertainty.

Interleaving is not a cure for all interference or thermal drift.

Exercise the actual integrated path, including dispatch, fallback and relevant forward/backward
behavior. Verify correctness against the contract over representative, boundary and applicable
adversarial inputs; include repeated execution, state reset, concurrency or synchronization cases
when relevant.

Confirm the optimized implementation is actually called when the contract requires it. Preserve the
baseline on failure. Integration must be remeasured even if a standalone kernel passed.

Separate smoke tests and partial workload samples from the complete acceptance matrix. Incomplete,
timed-out or failed cases are not passing results.

Before reusing results or accepting a resumed run, compare the relevant code, built artifacts,
harness, inputs and contract with the recorded identities; changes can invalidate prior evidence.
Run baseline and candidate without competing GPU experiments when practical, or record contention
and treat affected measurements as inconclusive.

**Example:** A candidate saves a forward activation to avoid backward recomputation. Measure the
selected memory metric over the complete forward-to-backward lifetime and compare the integrated
forward+backward latency against the same workload's baseline. A backward-only timing or the size of
the saved tensor alone cannot establish either acceptance condition.
