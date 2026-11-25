# Whole-path diagnosis and architecture search

Use the parts that help explain the current result or select the next experiment. Match the depth
of diagnosis to the decision, scope and budget.

## Map the work

Record the end-to-end baseline for each representative workload when available; for a new operator
without one, record that gap and map the reference path or available estimates.

Build a compact stage map with stage/dispatch, operation or kernel, input shape, time or
critical-path contribution, memory lifetime, overlap/dependencies, evidence and uncertainty. Include
setup, preprocessing, communication and postprocessing when inside the user's boundary, along with
main kernels and relevant forward/backward.

If a stage cannot be isolated, mark it unknown and identify the cheapest measurement that would
change the decision.

Rank opportunities by expected end-to-end impact and cost, not kernel duration alone.

Distinguish CPU and GPU time and account for overlapping work; summed GPU durations are not
necessarily wall time.

As a sanity bound, an ideal elimination of a non-overlapping stage occupying fraction `f` of elapsed
time cannot improve whole-path latency by more than `f`; this bound does not apply unchanged when
dependencies, overlap or the algorithm change. Use actual workload frequency or critical-path data
where available and report per-case regressions.

### Diagnose headroom inside fused kernels

When a fused kernel contributes material latency, use source and dispatch to map the internal
stages that could explain it. Mark each relevant stage's cost as measured, bounded or unknown.
Choose a matched pair of workload cases that changes the work in the suspected stage while
holding other inputs and settings steady where practical. Record the actual dispatch for both
cases. Use the pair to select a stage to investigate, since a mode change can also alter scheduling
or overlap.

Before deprioritizing a stage after a local candidate fails, record the exact hypothesis that
the result ruled out or left unresolved. If the stage could still contribute meaningful latency,
select the cheapest discriminating check, such as targeted instrumentation, a focused component
benchmark or a mechanism replacement that preserves the surrounding work. For example, in fused
backward, compare main-gradient accumulation with overwrite, then test the accumulation mechanism
with the same routing, communication and matrix multiplication where practical. A wider global
store tests one implementation choice. An alternate read–accumulate–write path tests a different
mechanism.

Record the stage evidence, what it says about remaining headroom and the next decision. If the
check is unavailable or disproportionate to the task, record the unresolved headroom and the
reason for deferral. Confirm any gain with uninstrumented timing at the task's measurement boundary.

## Compare directions

When choosing between promising alternatives, consider:

- Algorithm and execution structure: stage and fusion boundaries, layout/data movement, parallel
  decomposition, scheduling, persistent versus non-persistent execution and dispatch as applicable.
  A different tile size alone is normally a local variant, not a new architecture.
- Claimed bottleneck and evidence; the expected whole-path benefit or upper bound and cases where it
  matters. Label analytic estimates, profiler observations and measured results separately.
- Memory lifetime and peak estimate, correctness and interface consequences, affected components,
  risks and minimum experiment capable of falsifying the hypothesis.
- Choice and reason: selected, deferred or infeasible. A reasonable cross-stage option can be a
  split, fusion, alternate routing or scheduling, or a saved-state/recompute choice; respect
  explicit constraints and do not force an unsuitable design.

For an intended mechanism, predict an observable direction against a comparable baseline, such as
less local-memory traffic for a spill-reduction hypothesis. Use available counters, compiler reports
or a focused microbenchmark. A counter is evidence, not automatically a cause or the objective; do
not require a specific instruction merely to imitate a reference. If performance improves while the
mechanism is unconfirmed, retain the result, revise the explanation and identify a discriminating
follow-up.

When external implementations can inform a direction, select references by operator, hardware and
implementation language. Record the relevant entry point, revision and applicable design idea with
its source; check workload and version differences before using its claims. Consult references as
needed instead of cloning a fixed set for every task.

## Choose the next experiment

Balance the remaining end-to-end headroom, how much a result could change the next decision, and
what the experiment costs. If local variants start repeating the same question, consider a cheap
test of another mechanism even while the current direction yields gains. Follow strong evidence
directly when the next experiment is clear.

## Reassess and choose the next search

Set review checkpoints appropriate to the task's time and experiment budget. Reassess when repeated
distinct attempts do not yield a meaningful improvement beyond noise, the decision space for the
current approach is exhausted, the critical path changes, correctness/debugging stalls, or observed
behavior contradicts the model. Do not use a universal fixed number of attempts.

At a checkpoint, revisit the best validated result, the evidence for the current bottleneck and
the remaining headroom. Choose an experiment that resolves a consequential uncertainty or record
why the direction is being deferred. Revisit untested alternatives when they could materially
affect the outcome.

For long searches where multiple structural directions are feasible and the expected value justifies
the cost, keep a small portfolio: for each architecture record its defining difference, best valid
candidate, current evidence, next discriminating experiment, resources spent and selection/deferment
reason. Treat tile-only variants as part of their parent architecture unless evidence shows a
structural change. Allocate exploratory work within any user-specified budget, periodically compare
the expected value of the next experiment per direction, and record why a direction is continued or
retired.

If no budget was specified, track cost and review points without inventing a hard limit or declaring
success when an arbitrary count is reached.

A fresh diagnosis can start from the contract, verified evidence and best implementation. Consult
earlier experiments when they bear on the new idea.

**Coupled changes:** Test dependent changes together when the mechanism requires it. For example,
test a larger tile with its matching work partition if the tile alone would hurt occupancy.
Record the dependency and evaluate the combined result against the task contract.

## Further reading

- [ForgeKernel milestone
  template](https://github.com/OpenBMB/ForgeKernel/blob/main/docs/milestones-template.md): mechanism
  checks and coupled decisions.
- [CAKE](https://arxiv.org/html/2608.12629): structured compiler feedback and dispatch-aware
  generalization.
- [Evolving Agent Teams](https://arxiv.org/html/2610.36848): diagnosing search stagnation and
  preserving experience.
- [KernelOPT](https://arxiv.org/html/2609.30059): diverse candidate search and end-to-end
  verification; its reported search-time correctness flag is not used to filter beam ranking.
- [autokernel-skills](https://github.com/reyoung/autokernel-skills/tree/f37bd4d02eca8c0e71124a10c25a1a95a091b29c):
  operator-specific references, benchmark construction and experiment context.
