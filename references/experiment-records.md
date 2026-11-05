# Experiment records and decisions

Use existing project formats when they capture this evidence.

Otherwise keep a concise plan and append-only candidate/measurement log in the task workspace. The
point is to reconstruct a decision, not to create a new harness or a prescribed file tree.

## Minimum records

- **Contract and baseline:** task objective; workload cases and holdout if needed, including what
  the optimizer could see and what influenced tuning; correctness and resource limits; timing and
  memory scope, tools and blind spots; source/build and actual binary identity, evaluator/oracle and
  input/configuration identity, hardware/software versions, execution conditions, commands and
  baseline samples; acceptance rule. Use project fingerprints or hashes where available, including
  uncommitted source changes; distinguish verified integrity from a procedural check.
- **Stage map and plan:** critical-path evidence and uncertainty; architecture alternatives,
  estimates and reasons for selection/deferment; falsifiable mechanisms and smallest tests; exact
  correctness and benchmark commands; relevant external implementation and revision when used.
- **Candidate:** ID and parent or recoverable code identity; intended change or coupled group;
  workload, expected mechanism, memory estimate, risk and safeguards/coverage; correctness and
  negative-control evidence; observed mechanism and limitations; memory per case/device; timing
  samples, variability estimate and per-case comparison; status and decision reason. Link raw
  reports when used.
- **Reassessment:** trigger, current best, supported and rejected hypotheses, remaining
  alternatives, next experiment and budget. For long multi-architecture searches retain the
  portfolio described in [architecture search](architecture-search.md). Mark observations, estimates
  and unresolved claims distinctly.

A compact log entry can use:

```text
candidate: <id; parent; code/build identity>
change / hypothesis: <mechanism, expected evidence; coupled steps if any>
contract: <workload; timing and memory scopes; limits>
evidence: <correctness; memory per device; baseline/candidate samples; mechanism observation; report links>
decision: <keep exploring | ready for integrated acceptance | promote | reject | unresolved; reason and next step>
```

## Decision rules

1. **Correctness:** the intended integrated path meets the numerical and behavior contract; the
   validator's negative control fails as intended. Missing or invalid evidence is unresolved, not a
   pass.
2. **Resources:** apply the hard-limit or risk-based procedure in
   [measurement and validation](measurement.md). For each case and device, compare measured memory
   against the applicable peak limit or allowed increase over baseline. Record measurement and
   protection gaps; do not claim that an uncovered scope or required criterion passed. Report
   acceptance only for the measured scope and its stated blind spots.
3. **Performance:** compare the stated end-to-end target and, where applicable, per-case regressions
   under comparable conditions. Compare relevant objective metrics with both the initial baseline and
   current best when available; apply the contract's tradeoff rules. A baseline-free task with an
   explicit absolute target can be evaluated against that target, but cannot claim baseline-relative
   improvement. Establish per-case variability before judging a gain; without it, the apparent
   improvement is uncertain. Apply the
   user's or project's estimator, tolerance and acceptance rule; a noise-sized difference remains
   uncertain unless the rule establishes otherwise. Profiling explains a result but does not replace
   uninstrumented acceptance timing.
4. **Integration:** verify that dispatch uses the intended candidate, and rerun the checks above
   after integration with the evaluator boundary in [measurement and validation](measurement.md).
   If the intended integration base has advanced since the experiment began, re-evaluate the
   candidate against the updated base before promotion; preserve both candidate and current best
   until then.
   Report holdout visibility and whether integrity was enforced or procedural; do not label an
   exposed holdout as hidden or a tuning-used holdout as independently held out. If no candidate
   meets the contract, retain the last valid implementation and report the blocker or next
   experiment; do not silently relax limits. If the fastest fails, another recorded candidate may be
   evaluated under the same contract.

## Interrupted or failed runs

Use project-provided timeouts and recovery procedures; otherwise choose bounded compile, execution
and profiling timeouts appropriate to the task. Record the command, case, stage, candidate identity,
logs and available job/process identifiers. Distinguish compile failure, numerical mismatch,
timeout/hang, out-of-memory, environment/transport failure and inconclusive evidence; do not label
an infrastructure failure as a correctness failure or a timeout as a measured latency.

For repeated failures, preserve a minimal reproducer, observed facts and the next diagnostic
hypothesis.

Before execution, identify the backend's supported stop and recovery procedure and record the
job/client IDs and PID namespace needed to target this run. After a hang or device fault, stop only
work owned by the run through the applicable execution system, preserve diagnostics, and use an
appropriate lightweight health check before further experiments.

If ownership, permissions or the supported control path are unclear, stop retries and report the
blocker. Do not reset a shared GPU or remove others' work. If health cannot be established, stop GPU
experiments and report the blocker.

If a remote submission's status is unknown, reconcile it before retrying to avoid duplicate jobs. On
resume, verify the recorded code, binary, harness, input and contract identities; rerun invalidated
evidence instead of combining incompatible results.

**Illustrative result:** For a baseline peak of 10 GiB and a user-allowed increase of 5%, the
acceptance threshold is 10.5 GiB on the same workload and scope. A measured 10.4 GiB can pass that
check; an 11 GiB candidate needs an authorized tradeoff before promotion. Both still require
correctness and whole-path performance checks. An applicable absolute limit also takes precedence.
