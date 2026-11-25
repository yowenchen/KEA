# Measurement and validation

Use this reference for the contract, baseline, candidate comparison and acceptance. Prefer the
project's existing harness and acceptance policy; record deviations and measurement blind spots.

A measurement excludes preparation from latency only when the task's actual timing contract does so;
the resources held by preparation may still count toward peak memory.

## Freeze the contract

- Identify the tested build, hardware, workload and execution mode. Record the relevant input
  details and any weighting or per-case regression limits. Use the holdout guidance when claiming
  generalization from tuned cases or when the task requires it.
- Define which work is timed and which allocations and lifetimes count toward peak memory.
  Record warmup, repetitions, estimator, noise treatment, reset requirements and exact commands.
  Record user-specified time, GPU, iteration or cost limits and stop conditions when provided.
  Define each decision metric's unit, direction and aggregation. Count setup, compilation and
  validation toward a wall-clock budget when the user's budget includes them. Preserve the
  acceptance inputs, reference and thresholds. If they change, rerun the affected comparisons.
- Establish the correctness reference from the specification, a trusted independent implementation
  or, when the contract requires behavior preservation, the frozen baseline. It must not call the
  candidate as its oracle. Reuse trusted validation evidence that covers the affected behavior.
  Check a new or materially changed validator with a representative controlled error, then restore
  the normal inputs. An invalid or unavailable oracle leaves correctness unresolved.

### Holdout and evaluator boundary

For an independent holdout, reserve cases or a sampling rule before tuning. If a holdout is added
later, freeze the candidate and select fresh cases independently of its results. Record whether
holdout information influenced tuning and whether its cases, inputs and results were hidden from
the optimizing agent. A request to avoid inspecting them provides procedural separation only.

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
source/build identities match the frozen contract and that the required cases ran.

Preserve original evaluator outputs; a candidate's own summary is not a substitute. If tests or the
evaluator must change, record why and re-establish the affected baseline and evidence; honor
user-authorized test changes.

Where the same agent can alter both the evaluator and records without enforcement, describe this as
a procedural check, not tool-enforced integrity. Do not create a new isolation service solely to
satisfy this step.

## GPU memory

Record where each memory constraint applies. Treat a stated memory budget as a delivery target
unless the task or environment applies it to execution. Apply hard limits to the runs they cover.
Use the current task's target and any user- or project-authorized tolerance. Choose a matching
metric and state its device/process scope, units and coverage.
Without a specified scope, prefer a relevant process- or device-wide metric when available; otherwise
use the best available allocator metric and record external-allocation blind spots.
Measure the full relevant lifetime and include allocations that are already live when measurement
begins. Count inputs, outputs, retained state, scratch space and temporary peaks as specified in
the contract. Only combine component allocations that are live at the same time.

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
- Apply a relative target to the comparable baseline peak for each workload and device,
  using the same selected metric and scope. A percentage applies to each baseline; an absolute
  increment is added to it. Apply applicable targets and execution limits independently.
  An increment needs a baseline; if none exists, request an absolute criterion rather than inventing
  a relative result.
- If the user delegates the tradeoff, choose and record a tolerance justified by the workload
  and device headroom. Use an absolute criterion when the baseline is zero.
- Repeat comparable measurements when a small overage could be explained by measurement noise.
  Report the measured delta, uncertainty and performance benefit. For promotion, apply the target
  with any stated tolerance or delegated judgment. If the result falls outside that scope, keep it
  exploratory or seek agreement on the specific tradeoff. A small overage alone does not establish
  a universal tolerance.
- If measurement or baseline coverage is incomplete, record the gap. Do not claim a memory result
  beyond the measured scope or that a required criterion passed. Without a stricter user or project
  requirement, a result may be accepted for the measured scope when inspection and available
  evidence support no material increase in unmeasured allocations. Otherwise the memory result is
  unresolved. Disclose blind spots and do not claim total GPU memory stayed within the tolerance.

### Before candidate execution

Identify the execution host, devices, existing limits and measurement coverage. Estimate the change
in peak live memory over the full affected lifetime, including saved tensors, workspace and external
allocations; do not assume that unchanged tensor sizes imply unchanged peak lifetimes. Check current
available memory and expected headroom, noting other users and allocation uncertainty.

Apply any [hard limit](#hard-limits-and-protection) before running. Choose safeguards proportional
to the candidate:

- **Low risk:** for a change with no material new allocations or lifetime extension, run normal
  staged tests when the estimate and available headroom support it. Measure peak memory and compare
  it with the baseline; a verified process-wide hard limiter is not required.
- **Higher or unknown risk:** examples include a large or input-dependent workspace, newly retained
  state, or opaque allocations. Estimate size and lifetime, use an existing effective
  limit where available, and start with a bounded smaller case when it can safely distinguish the
  hypothesis. Check the observed peak against the estimate before scaling. Bound memory separately
  for larger cases if their allocation scaling or implementation is uncertain. When no baseline
  exists, use available reference measurements and device headroom to assess risk, and report
  that a baseline-relative screen is unavailable.

If no delivery allowance is specified, a bounded screen can proceed when the candidate's memory
risk is understood and available headroom and hard limits permit. Ask about the tradeoff when the
answer would affect the scope of the search or the delivered choice. Record the measured change
and seek agreement before accepting a material increase with no agreed allowance.

Apply the risk assessment to warmup, smoke, correctness, profiling and acceptance runs. Record the
estimate, relevant headroom, measurements, safeguards and blind spots. A peak reading or sampled
monitor detects some excess only after allocation; it does not enforce a limit or guarantee that a
short-lived peak was observed. If the observed peak invalidates the estimate used to establish
safe headroom, reassess before scaling.

### Recover memory on a promising candidate

When a correct candidate delivers a substantial measured improvement while exceeding a memory
target, retain it and the best in-target candidate if one exists. If hard limits and device
headroom permit, use the improved version as a parent for further performance work. Keep its
code identity, results and measured overage recoverable. Pursue memory recovery once the
performance direction is established, or sooner if headroom would block the next useful experiment.

Identify the allocations live at peak and quantify how many bytes to recover. Test feasible
changes such as shortening a buffer lifetime, reusing workspace or trading storage for
recomputation. Compare peak memory and the task metric on the same workload against the parent
and the best in-target alternative when available. Recheck both after integration.

If feasible recovery work within the task's scope and budget leaves the result above the allowed
peak, request user review. Include its code identity, performance benefit, measured peak and
overage, the recovery attempts and outcomes, and the best in-target alternative when available.
Keep both available for the decision.

### Hard limits and protection

Apply an absolute hard memory limit to baseline, warmup, negative-control and candidate
execution. If an allowed increment must also never be exceeded during execution, apply its derived
limit to candidate runs. Before the affected execution, use a verified runtime limit or isolation
mechanism covering the required devices, processes, allocation types and lifetime, or a conservative,
justified upper bound on simultaneous live allocations for that exact path that does not exceed the
limit. A rough estimate, post-run peak or sampling monitor alone is not enforcement. If neither
option covers the required scope, do not run the affected execution; record the gap and continue
safe investigation.

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

## Screen the mechanism on informative cases

Start with a case where the hypothesized effect should be visible and, when available, a matched
control where it should be weaker or absent. Use code and available measurements to choose the
cases. Keep input distribution, execution mode and other settings matched where practical,
and verify which implementation ran in both cases. If changing the mode also changes scheduling or
overlap, record that confound and use a focused check if it affects the decision.

Pair baseline and candidate within each case. Record the per-case result, variability and whether it
supports the predicted effect. Expand coverage when the signal warrants it. Repeat a noisy screen
when resolving it would change the next decision. Use the full task contract for acceptance.

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

For short GPU kernels, check whether CPU submission gaps leave the GPU idle between timed calls when
that would distort the intended metric. Use a workload-compatible measurement method and confirm its
timing boundary. Graph replay, batching or queued work may change launch overhead, cache, clocks or
execution semantics: use them for acceptance only when they match the contract; otherwise label them
diagnostic and retain the contract's measurement for the result.

Exercise the workload through its normal entry point, including any relevant fallback behavior.
Verify correctness against the contract over representative, boundary and applicable
adversarial inputs; include repeated execution, state reset, concurrency or synchronization cases
when relevant.

Confirm the measured path used the optimized implementation. If the intended workload spans
surrounding operations, verify it there even if the standalone result passed.

Treat smoke and partial results as exploratory. Count completed, verified cases toward acceptance.

Where practical, verify outputs or state from the actual timed executions outside the timing boundary,
especially for repeated or stateful calls. For matching cases, keep verification and benchmark inputs
and implementation consistent. Record intentional differences. Preserve required inputs and outputs
without changing the measured resource contract; if outputs are overwritten or retaining them would
alter memory use, adapt the harness or verify a representative subset and record that coverage. A
separate rerun does not prove that the timed calls were correct. If the contract requires checks that
cannot be performed, mark acceptance unresolved.

Before reusing results or accepting a resumed run, compare the relevant code, built artifacts,
harness, inputs and contract with the recorded identities; changes can invalidate prior evidence.
Run baseline and candidate without competing GPU experiments when practical, or record contention
and treat affected measurements as inconclusive.

**Example:** If a candidate keeps an intermediate to speed up a later operation, measure latency
across the work affected and peak memory over the intermediate's lifetime. Compare both with the
same workload's baseline.
