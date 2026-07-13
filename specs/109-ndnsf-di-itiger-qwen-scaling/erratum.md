# Spec 109 formal erratum: live iTiger inference was not executed

**Issued**: 2026-07-13
**Applies to**: Spec 109 documents and terminal campaign records
**Historical commit**: `8b9a4fe709d35b9e4d4961eaa25cefad45cfc0b2`
**Correction owner**: `specs/110-itiger-qwen-live-inference/`

## Corrected statement

Spec 109 completed implementation scaffolding, storage discovery, 0.5B model
transfer/sealing, validation logic, negative-result accounting, and audit
closure. It did **not** execute Qwen inference on an iTiger GPU. Slurm jobs
146050 and 146123 transferred and sealed the 0.5B model; neither was an
inference job. The number of GPU inference jobs was zero.

Consequently, `165/165 tasks complete` must be read as **terminal accounting
complete**, not **experiment objective complete**. No TTFT, inter-token latency,
tokens/s, request throughput, GPU-memory peak, or size-scaling result was
measured.

## Cause

The Spec 107/108 gates correctly identified that the NDNSF-DI generation
session, GPU release, and multi-node NFD substrate were not ready. The defect
was allowing those pre-start blockers to close tasks whose wording required a
real Slurm inference submission. This converted a legitimate readiness failure
into apparent experiment completion and obscured the user's actual objective:
run Qwen through the NDNSF-DI distributed request, security, provider,
dependency, GPU, and response path on iTiger.

## Non-destructive correction

- Do not rewrite, delete, or promote any Spec 109 result.
- Do not reuse a Spec 109 candidate, campaign, cell, run, or once-only Slurm
  submission identity.
- Do not change `release-gate.json`; its `BLOCKED` verdict remains the correct
  verdict for the historical campaign.
- Use standalone Qwen inference only as an independently runnable correctness
  and capacity baseline; it does not satisfy the requested experiment.
- Treat missing Spec 107/108 capabilities as work to finish or repair, not as
  terminal satisfaction of a Spec 110 distributed-inference task.
- Execute the corrected live matrix only under Spec 110.

## Completion-semantics correction

For Spec 110, a task that says **run**, **submit**, **measure**, or **reproduce**
cannot be checked merely because a pre-start `BLOCKED`, `DEFERRED`, or
`NOT_STARTED` record exists. It closes only after the required real inference
attempt reaches its defined execution boundary and durable evidence is
promoted. Capacity or quota shortage creates a remediation task and leaves the
live inference task open.

An execution failure after the model process begins—such as load failure, CUDA
failure, OOM, timeout, or incorrect tokens—is a valid measured negative result.
A scheduler, VPN, quota, image, or storage failure before inference begins is an
operational blocker, not a completed inference experiment.

## Successor boundary

[Spec 110](../110-itiger-qwen-live-inference/spec.md) owns the corrected iTiger
experiment: provision the complete NDN/NDNSF/NDNSF-DI/Qwen GPU environment,
prove allocation-scoped multi-node NFD connectivity, and execute real
distributed Qwen2.5-Instruct candidates for 0.5B, 1.5B, 3B, 7B, 14B, 32B, and
72B. Standalone inference is retained only as the matched oracle. A Spec 110
distributed task stays open until the real NDNSF-DI candidate reaches its
defined execution boundary.
