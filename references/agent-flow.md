# KEA agent flow

Follow this loop in the implementation workspace. Reuse the project's plans, harnesses and evidence
records when suitable; [experiment records](experiment-records.md) defines their minimum contents
without requiring particular filenames. The user and project determine scope, runtime conditions and
acceptance rules.

## 1. Contract and baseline

Read [measurement and validation](measurement.md). Inspect the task, implementation, tests and
project instructions. Record the workloads and representative cases, inputs/outputs, correctness,
allowed changes, hardware, timing and memory boundaries, constraints, commands and acceptance
criteria. Include the full forward-to-backward lifetime when backward is in scope. If no memory
budget is specified, use the comparable baseline peak for each workload as its default ceiling, as
specified in the entrypoint.

Before baseline or negative-control GPU execution, apply any explicit memory limit using the
discovery and protection procedure in the measurement reference. When the default ceiling depends on
the first baseline, record that the ceiling is not yet established while measuring it.

Measure a comparable baseline when available and check that the correctness validator rejects a
representative controlled error; restore temporary changes. Record code/build identity, measurement
conditions and raw evidence.

If the baseline, validator or memory coverage is unavailable, record what is unknown and continue
safe inspection or planning; do not claim that the corresponding acceptance criterion passed.

A new operator without a performance baseline may still be accepted against explicit absolute
performance and memory criteria, provided those criteria and correctness can be verified
independently; a comparative claim or default baseline-derived ceiling remains unavailable.

**Ready for diagnosis:** the contract, measurement boundaries, baseline evidence and gaps are
recorded. If the default ceiling has not been established, do not run an experiment whose memory use
cannot be kept within an applicable limit.

## 2. Whole-path diagnosis and plan

Read [architecture search](architecture-search.md). Map the user-facing path, including dispatch,
preprocessing, main kernels, postprocessing and relevant forward/backward or communication. Record
critical-path evidence, overlaps, uncertainty, expected whole-path benefit and important
outside-scope bottlenecks.

Write a short plan before editing implementation code. Compare the current architecture with
feasible alternatives, including a local improvement and a materially different cross-stage or
structural direction when applicable. For each promising option record the expected benefit, memory
and correctness implications, risks, falsifiable mechanism, cheapest discriminating experiment and
exact validation and measurement commands.

Explain an infeasible alternative rather than implementing it mechanically. Capture component
layout, synchronization, ownership and lifetimes where relevant.

**Ready for implementation:** the stage map and plan identify the first hypothesis, evidence needed
to test it and how to keep the current best recoverable.

Unknowns and unavailable measurements have explicit next steps or are recorded as limitations.

## 3. Implement and test a candidate

Read [experiment records](experiment-records.md). Identify the candidate and parent and establish
the runtime memory protection in [measurement and validation](measurement.md) before any candidate
GPU execution, including warmup, smoke and correctness checks. Record its coverage and the
candidate's memory estimate. Test one hypothesis at a time; a hypothesis may contain coupled
changes. Record their dependency, expected intermediate regressions and group-level test before
implementing them.

Run checks in stages: compile and launch smoke, correctness, then memory and uninstrumented
performance when it is safe and meaningful to do so. If a check fails or a resource estimate exceeds
the limit, do not run later stages merely to fill in metrics; record them as not run or unresolved
and proceed to a decision. Smoke checks do not replace the full acceptance suite.

For key claimed mechanisms compare the predicted evidence with the observation. If a speedup occurs
without the predicted mechanism, keep the measured result, mark the explanation unconfirmed and
investigate.

Record unavailable measurements rather than inventing evidence.

**Ready for a decision:** record the candidate and recoverable code identity, checks actually run,
their conditions and results, and why any required evidence is missing. A failed or unmeasurable
candidate can inform investigation but cannot be accepted; a candidate eligible for acceptance needs
complete contract evidence.

## 4. Decide and evolve

Use the decision rules in [experiment records](experiment-records.md) to classify the candidate as
keep exploring, ready for integrated acceptance, reject or unresolved, with a reason. Retain a
non-winning intermediate architecture when a focused experiment can test its potential. Preserve the
best valid candidate and the experiment history; promotion follows integrated acceptance.

When progress plateaus, the bottleneck changes or measurements contradict the model, use the
reassessment in [architecture search](architecture-search.md): revisit the full path, supported
assumptions, headroom and feasible structural alternatives; update the plan and select the next
discriminating experiment.

For longer searches where the budget justifies it, retain a few structurally distinct candidates
and, on stagnation, reassess from a concise record before cross-checking against the full history.
Preserve the contract and complete audit trail.

**Next step:** either return to candidate implementation with a recorded hypothesis, prepare an
eligible candidate for integrated acceptance, or report a concrete blocker. Do not silently relax
the user's scope or resource constraints.

## 5. Integrated acceptance and report

Use the measurement and records references to verify the actual call path runs the intended
candidate and recheck correctness, memory and end-to-end performance at the contract boundary.
Record any allowed fallback and the path measured.

Compare representative workloads and individual regressions; use held-out workloads when shape
generalization is in scope.

Establish measurement variability before assessing a gain; for gains near noise, repeat interleaved
baseline/candidate measurements when practical and report unresolved differences as uncertain.

**Complete:** the final candidate meets the correctness, memory and performance acceptance criteria
in the contract, with evidence tied to the final code and configuration. Review the promising
structural alternatives as described in [architecture search](architecture-search.md): record a
discriminating validation or an evidence-backed reason to defer each applicable direction.

Record the selected result, tradeoffs, rejected alternatives, assumptions and limitations.

If no candidate qualifies, retain the last valid implementation and report the blocker or next
experiment; another recorded candidate can be evaluated under the same contract. Distinguish an
achieved objective from an exhausted user-specified budget or external blocker; neither incomplete
evidence nor an arbitrary iteration count is success.
