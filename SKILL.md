---
name: kea
description: Apply Kernel Evolution Agents (KEA) when asked for KEA or an evolutionary GPU kernel optimization workflow that explores whole-path bottlenecks and architecture changes while respecting resource constraints, including when no memory budget was specified.
---

# Kernel Evolution Agents

Evolve the implementation against the user's actual workload, end-to-end objective and resource
constraints.

The user's requirements and project rules take precedence. Put task artifacts in the implementation
workspace, following existing conventions; do not modify this installed skill during a task.

## Run the workflow

Read [agent flow](references/agent-flow.md) and follow its stages, evidence requirements and
decision points. It links the measurement, architecture search and experiment-record references
needed at each stage.

The user's or project's memory limit and measurement scope apply. If the user has not specified an
allowed memory increase or equivalent task-specific peak target, ask early how much peak GPU memory
may increase (absolute amount or percentage). A platform capacity limit alone does not express the
user's desired tradeoff. While awaiting the answer, continue safe work that does not depend on it;
do not assume consent or run candidate GPU experiments. Apply the answer against each workload's
comparable baseline as described in
[measurement and validation](references/measurement.md).

Account for saved tensors and temporary workspace over their lifetimes, including relevant
allocations outside the framework allocator.

Before candidate execution, assess incremental memory risk and follow the measurement reference.
Enforce hard limits as specified there; otherwise proportionate checks and measurement allow
ordinary low-risk experiments without a hard limiter.

When required protection, baseline or memory measurement is unavailable, record the gap and continue
safe investigation; do not claim the corresponding criterion is met or promote an unverified
candidate.

[KernelWiki](https://github.com/mit-han-lab/KernelWiki) and
[ncu-report-skill](https://github.com/mit-han-lab/ncu-report-skill) are optional companion skills;
the agent flow specifies when to use them and what to do when they are unavailable. Component
contracts do not replace correctness and synchronization testing.
