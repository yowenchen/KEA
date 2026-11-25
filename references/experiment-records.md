# Experiment records and decisions

Use the project's records or a short experiment log in the task workspace. Record shared setup,
constraints and baseline evidence once, then link to them. For each experiment that affects a
decision, capture what changed, which cases ran, what was observed and what you decided. A small
experiment can fit on one line. Mark estimates and uncertain results as such.

For results used to select a candidate or claim acceptance, retain the recoverable code and build
identity, workload and environment, benchmark and validation commands, raw evidence, and any
measurement gaps. Use the [measurement guidance](measurement.md) for the evidence relevant to
the claim.

A compact log entry can use:

```text
<change or candidate ID> | <cases> | <result and evidence link> | <decision and reason>
if no gain: <what the result rules out and whether to keep, defer or close the direction>
```

## Decision rules

1. **Correctness:** the intended integrated path meets the numerical and behavior contract. Use
   trusted or independently checked validation evidence as described in
   [measurement and validation](measurement.md#freeze-the-contract).
2. **Resources:** apply the hard-limit or risk-based procedure in
   [measurement and validation](measurement.md). For each case and device, compare measured memory
   against the target and any agreed tolerance, and verify applicable hard limits. Keep promising
   over-target candidates eligible for [memory recovery](measurement.md#recover-memory-on-a-promising-candidate).
   Record measurement and protection gaps. Report acceptance only for the measured scope and its
   stated blind spots.
3. **Performance:** compare the stated end-to-end target and, where applicable, per-case regressions
   under comparable conditions. Compare relevant objective metrics with both the initial baseline and
   current best when available; apply the contract's tradeoff rules. A baseline-free task with an
   explicit absolute target can be evaluated against that target, but cannot claim baseline-relative
   improvement. Establish per-case variability before judging a gain; without it, the apparent
   improvement is uncertain. Apply the
   user's or project's estimator, tolerance and acceptance rule; a noise-sized difference remains
   uncertain unless the rule establishes otherwise. Profiling explains a result but does not replace
   uninstrumented acceptance timing.
4. **Integration:** verify that the measured path uses the intended candidate. Rerun the checks
   that apply to the integrated workload using the agreed [measurement boundary](measurement.md).
   If the intended integration base has advanced since the experiment began, re-evaluate the
   candidate against the updated base before promotion; preserve both candidate and current best
   until then.
   Report holdout visibility and whether integrity was enforced or procedural; do not label an
   exposed holdout as hidden or a tuning-used holdout as independently held out. If no candidate
   meets the contract, retain the last valid implementation and report the blocker or next
   experiment; do not silently relax limits. If the fastest fails, another recorded candidate may be
   evaluated under the same contract.

## Interpret a no-gain result

Apply this at candidate decision time when a screen shows no meaningful gain, a regression, or
an inconclusive result.

1. Verify that the tested build and workload exercised the intended change. Check whether
   measurement variability can explain the result. Mark inconclusive observations and choose a
   repeat or correction when it could change the decision.
2. Compare the predicted local effect with the evidence. If the effect was absent, identify what
   in this implementation could explain it. If the effect occurred without improving the task
   metric, check whether it affects that metric and whether changes elsewhere offset the benefit.
   Mark explanations without evidence as hypotheses. Choose a focused follow-up when its result
   could change the next decision.
3. Record what the result rules out and decide the direction separately. Close a direction only
   when the evidence addresses the shared mechanism, such as a measured upper bound too small to
   matter or a demonstrated conflict with the task constraints. Otherwise keep it open or defer
   it with a reason and a condition for revisiting it. A local rejection can coexist with an
   open direction. Use the available budget to decide whether to run the follow-up now.

## Interrupted or failed runs

Use project-provided timeouts and recovery procedures; otherwise choose bounded compile, execution
and profiling timeouts appropriate to the task. Record the command, case, stage, candidate identity,
logs and available job/process identifiers. Distinguish compile failure, numerical mismatch,
timeout/hang, out-of-memory, environment/transport failure and inconclusive evidence; do not label
an infrastructure failure as a correctness failure or a timeout as a measured latency.

For repeated failures, preserve a minimal reproducer, observed facts and the next diagnostic
hypothesis.

For remote or long-running work, identify how to stop the run and retain the relevant identifiers
once available. After a hang or device fault, stop only work owned by the run, preserve diagnostics,
and use an appropriate lightweight health check before further experiments.

If ownership, permissions or the supported control path are unclear, stop retries and report the
blocker. Do not reset a shared GPU or remove others' work. If health cannot be established, stop GPU
experiments and report the blocker.

If a remote submission's status is unknown, reconcile it before retrying to avoid duplicate jobs. On
resume, verify the recorded code, binary, harness, input and contract identities; rerun invalidated
evidence instead of combining incompatible results.
