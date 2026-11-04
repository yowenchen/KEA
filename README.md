# KEA: Kernel Evolution Agents

KEA is a Codex skill for evidence-driven GPU kernel optimization. It treats the kernel as part of a
larger execution path, tracks memory as well as latency, and revisits architectural choices when
local tuning reaches its limits.

## Install

Clone the repository into your Codex skills directory:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
git clone https://github.com/yowenchen/KEA.git "${CODEX_HOME:-$HOME/.codex}/skills/kea"
```

The destination must not already exist. Restart Codex if needed for the new skill to appear.

## Use

Invoke `$kea` in a Codex conversation and describe the workload, target hardware, correctness
requirements, and any latency or memory constraints. For example:

```text
$kea Optimize this operator's forward and backward on H100. Keep peak GPU memory below 40 GiB
per device and evaluate end-to-end latency across the documented training shapes.
```

KEA follows an evidence-driven loop:

1. Define the task contract and measure a baseline.
2. Map the complete execution path and compare feasible architectural directions.
3. Test candidates, record results, and reassess when the bottleneck or evidence changes.
4. Accept a candidate only after integrated correctness, memory, and performance checks.

When no memory budget is specified, KEA uses the comparable baseline peak for each workload as a
conservative default. An explicit user or project limit takes precedence. Before running a
candidate, the workflow requires a verified limit or a justified upper bound that covers the
contract's allocation scope. If neither is available, GPU candidate execution remains blocked while
safe investigation can continue. The first baseline cannot be protected by a default ceiling that
has not yet been measured; explicit limits still apply to it.

KEA is a workflow, not a memory limiter or benchmark service. It relies on the target environment's
actual protection, profiling, tests, and evaluation tools. Framework allocator statistics and caps
may not cover all CUDA allocations. The task's existing correctness, warmup, preparation, and
measurement rules take precedence.

## Files

- [SKILL.md](SKILL.md): skill entry point and essential constraints.
- [Agent flow](references/agent-flow.md): stages, decisions, and completion criteria.
- [Measurement and validation](references/measurement.md): memory, benchmarking, and holdout
  procedures.
- [Architecture search](references/architecture-search.md): whole-path diagnosis and reassessment.
- [Experiment records](references/experiment-records.md): evidence, acceptance, and recovery.

## Optional companion skills

- [KernelWiki](https://github.com/mit-han-lab/KernelWiki) informs Blackwell/Hopper architecture
  choices during planning and reassessment.
- [ncu-report-skill](https://github.com/mit-han-lab/ncu-report-skill) diagnoses selected B200
  kernels after whole-path mapping and investigates candidate mechanisms when useful.

Install either skill alongside KEA if useful:

```bash
git clone https://github.com/mit-han-lab/KernelWiki.git "${CODEX_HOME:-$HOME/.codex}/skills/KernelWiki"
git clone https://github.com/mit-han-lab/ncu-report-skill.git "${CODEX_HOME:-$HOME/.codex}/skills/ncu-report-skill"
```

See each upstream repository for its requirements. Both are optional and are not bundled with KEA;
without them, use suitable hardware documentation and profiling tools.

## Validation status

The skill metadata and local references have been validated. A small H20 probe verified PyTorch's
caching-allocator limit, which does not establish a whole-process GPU memory limit. The complete KEA
workflow has not yet been validated end-to-end on a production kernel optimization task.
