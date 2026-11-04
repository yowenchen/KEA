# Whole-path diagnosis and architecture search

Use this reference before implementation and when progress stalls or the measured bottleneck
changes. Keep task-specific evidence in the workspace, not in the installed skill. Adapt the amount
of exploration to the user's scope and budget.

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

## Compare directions before tuning

For the current design and each promising alternative, capture:

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

## Reassess and choose the next search

Set review checkpoints appropriate to the task's time and experiment budget. Reassess when repeated
distinct attempts do not yield a meaningful improvement beyond noise, the decision space for the
current approach is exhausted, the critical path changes, correctness/debugging stalls, or observed
behavior contradicts the model. Do not use a universal fixed number of attempts.

At a checkpoint, record: current best and validated metrics; which bottleneck and assumptions the
evidence supports or contradicts; remaining headroom; whether attempted changes are only local
variants; at least one feasible structural alternative or why none is within scope; the next
discriminating experiment and expected cost. Update the plan or explain why the existing direction
remains best. Keep earlier candidate identities and failed experiments available.

Before declaring the task complete, review each promising in-scope structural alternative from the
plan. Record either a low-cost discriminating validation and its result, or a specific
evidence-backed reason to defer it, such as a measured bottleneck or upper bound, an incompatible
contract constraint, or disproportionate cost relative to the remaining user-specified budget. An
untested possibility is not a disproven design. Do not require implementing a structural alternative
that the evidence or task scope does not justify.

For long searches where multiple structural directions are feasible and the expected value justifies
the cost, keep a small portfolio: for each architecture record its defining difference, best valid
candidate, current evidence, next discriminating experiment, resources spent and selection/deferment
reason. Treat tile-only variants as part of their parent architecture unless evidence shows a
structural change. Allocate exploratory work within any user-specified budget, periodically compare
the expected value of the next experiment per direction, and record why a direction is continued or
retired.

If no budget was specified, track cost and review points without inventing a hard limit or declaring
success when an arbitrary count is reached.

A fresh diagnosis may temporarily omit stale speculation, but must retain the contract, verified
evidence and best implementation, then compare new ideas with the complete history before execution.
This can be done by one agent; delegation and parallel runs are conditional on available
authorization, tools and uncontended GPU resources.

**Coupled changes:** If a larger tile is expected to hurt occupancy until a matching work partition
is added, treat both as one hypothesis. Record the dependency, memory estimates and expected
intermediate regression before testing the group. Accept only the completed candidate against the
unchanged contract.

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
