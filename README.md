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
$kea Reduce end-to-end latency for this workload on the target GPU. Preserve the existing
numerical tolerances and report peak memory across the documented input sizes.
```

With a trusted baseline and a concrete hypothesis, a focused paired experiment can be a first
step. Choose planning, profiling, and architecture comparisons according to what the next decision needs.
Record consequential experiments and verify the final implementation on its actual call path for
correctness, memory, and performance.

A memory target guides delivery. A promising candidate above target can remain in the search while
peak memory is reduced. Bounded experiments can proceed when the memory risk is
understood, device headroom is sufficient, and applicable hard limits are satisfied. A hard
execution limit requires verified protection or a justified upper bound.

KEA is a workflow, not a memory limiter or benchmark service. It relies on the target environment's
actual protection, profiling, tests, and evaluation tools. Framework allocator statistics and caps
may not cover all CUDA allocations. The task's existing correctness, warmup, preparation, and
measurement rules take precedence.

## Files

- [SKILL.md](SKILL.md): skill entry point and essential constraints.
- [Agent flow](references/agent-flow.md): exploration decisions and acceptance criteria.
- [Measurement and validation](references/measurement.md): memory, benchmarking, and holdout
  procedures.
- [Architecture search](references/architecture-search.md): whole-path diagnosis and reassessment.
- [Experiment records](references/experiment-records.md): evidence, acceptance, and recovery.

## Optional companion skills

- [KernelWiki](https://github.com/mit-han-lab/KernelWiki) informs Blackwell/Hopper architecture
  choices during planning and reassessment.
- [ncu-report-skill](https://github.com/mit-han-lab/ncu-report-skill) provides targeted profiling
  guidance for B200 kernels.

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
