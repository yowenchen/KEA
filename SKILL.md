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

The user's or project's memory limit and measurement scope apply. If no budget is specified, use
each workload's comparable measured baseline peak as its default ceiling and state the assumption;
an explicit user constraint or tradeoff overrides it.

Account for saved tensors and temporary workspace over their lifetimes, including relevant
allocations outside the framework allocator.

Before running a candidate, establish the runtime budget protection specified in the measurement
reference; an estimate or post-run peak alone does not enforce a ceiling.

When required protection, baseline or memory measurement is unavailable, record the gap and continue
safe investigation; do not claim the corresponding criterion is met or promote an unverified
candidate.

For Blackwell/Hopper research, use the `KernelWiki` skill when it is installed; otherwise consult
appropriate hardware documentation.

For Nsight Compute on B200 / sm_100, use `ncu-report-skill` when it is installed. Otherwise, use
profiling guidance and metrics applicable to the hardware. Component contracts do not replace
correctness and synchronization testing.
