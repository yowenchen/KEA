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
per-case memory reference or explicit limit for holdout cases.

If an independent evaluator exists, let it retain those details and apply the appropriate per-case
memory checks and any required protection without exposing protected information to the search.
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
Without a specified scope, prefer a relevant process- or device-wide metric when available; otherwise
use the best available allocator metric and record external-allocation blind spots.
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
- Without an explicit budget, use the comparable baseline peak for each workload and device as a
  reference. By default, accept at most a 5% increase in the selected peak metric; this is a
  pragmatic tolerance, not a hardware safety guarantee. Record the assumption. User or project
  limits and tradeoffs override it. If the baseline is zero, allow no increase under this default.
- Repeat comparable measurements when the difference is near the tolerance or measurement noise.
  A candidate above the default tolerance can be explored when the risk procedure below permits,
  but do not promote it unless the user authorizes the tradeoff. Report both absolute and percentage
  changes, limitations and performance benefit. Estimates are not measurements.
- If measurement or baseline coverage is incomplete, record the gap. Do not claim a memory result
  beyond the measured scope or that a required criterion passed. Under the default policy, a result
  may be accepted for the measured scope when inspection and available evidence support no material
  increase in unmeasured allocations. Otherwise the memory result is unresolved. Disclose blind
  spots and do not claim total GPU memory stayed within the tolerance.

### Before candidate execution

Identify the execution host, devices, existing limits and measurement coverage. Estimate the change
in peak live memory over the full affected lifetime, including saved tensors, workspace and external
allocations; do not assume that unchanged tensor sizes imply unchanged peak lifetimes. Check current
available memory and expected headroom, noting other users and allocation uncertainty.

With no explicit limit, choose safeguards proportional to the candidate:

- **Low risk:** for a change with no material new allocations or lifetime extension, run normal
  staged tests when the estimate and available headroom support it. Measure peak memory and compare
  it with the baseline; a verified process-wide hard limiter is not required.
- **Higher or unknown risk:** examples include a large or shape-dependent workspace, new saved
  forward tensors, or opaque allocations. Estimate size and lifetime, use an existing effective
  limit where available, and start with a bounded smaller case when it can safely distinguish the
  hypothesis. Check the observed peak against the estimate before scaling. Do not extrapolate from a
  smaller case if allocation scaling or dispatch is unknown. As a default screening threshold, defer
  a full case estimated to exceed its baseline peak by more than 20% unless the user authorizes that
  experiment; reduce its memory use or test a smaller case first. If growth could cross that threshold
  and cannot be estimated with reasonable confidence, defer the run until it can be bounded or
  protected within the threshold. The 20% threshold screens experiment risk; it is not the 5%
  acceptance tolerance or a runtime cap. When no baseline exists, use available reference
  measurements and device headroom to assess risk, and report that a baseline-relative screen is
  unavailable.

Apply the risk assessment to warmup, smoke, correctness, profiling and acceptance runs. Record the
estimate, relevant headroom, measurements, safeguards and blind spots. A peak reading or sampled
monitor detects some excess only after allocation; it does not enforce a limit or guarantee that a
short-lived peak was observed. If a candidate exceeds the tolerance, investigate and reduce its
memory use or retain it as an exploratory tradeoff rather than silently promoting it. If an observed
peak exceeds the 20% screening threshold, stop scaling and reassess before further full-case runs.

### Explicit limits and protection

An explicit user or project memory limit applies to baseline, warmup, negative-control and candidate
execution. Before running, use a verified runtime limit or isolation mechanism covering the required
devices, processes, allocation types and lifetime, or a conservative, justified upper bound on
simultaneous live allocations for that exact path that does not exceed the limit. A rough estimate,
post-run peak or sampling monitor alone is not enforcement. If neither option covers the required
scope, do not run the affected execution; record the gap and continue safe investigation.

Inspect the actual target environment and effective limit, including device and process membership
and framework, library, custom and managed allocations. Use read-only status checks or an isolated
controlled probe within authorized boundaries. Do not alter shared GPU configuration just to discover
capabilities, fill a shared GPU, or assume an environment variable took effect. Record coverage and
blind spots; a framework allocator cap only covers its own allocations.

The limit's accounting scope can differ from the selected peak metric because of context, caches and
other allocations. If isolation changes visible compute or memory resources, obtain a comparable
baseline under those conditions and separately verify production timing. A user-authorized budget
or scope change requires a new contract; incomplete protection is not enforcement. Measure the
actual peak after execution for acceptance even when a limit was enforced.

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
