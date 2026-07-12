# Deployment Readiness Experiment Plan

## Material Passport

- Origin Skill: experiment-agent
- Origin Mode: plan
- Origin Date: 2026-07-12
- Verification Status: UNVERIFIED
- Primary Question: Can the fixed NDNSF-DI Qwen pilot execute real bounded
  generation correctly, predictably, securely, and recoverably under the
  declared deployment profile?

## Hypotheses and Falsifiers

| ID | Hypothesis | Primary metric | Falsifier |
|---|---|---|---|
| H1 | Evidence labels identify actual provider compute | provider/summary evidence agreement | any synthetic/mixed provider passes real gate |
| H2 | Three-stage Qwen output matches single node | token equality and numerical diagnostics | any unexplained token mismatch |
| H3 | Fixed 1 RPS pilot is serviceable | completion, achieved RPS, p95 ratio | <99% completion, <95% offered load, or >2x p95 |
| H4 | Measured telemetry makes plan validity safe | freshness/rejection decisions | stale/configured-only fact is accepted as measured |
| H5 | Dependency waiting is bounded | threads and retained state at 1,000 waits | threads scale with waits or state survives cancellation |
| H6 | One provider loss has bounded semantics | unique terminal outcome and recovery | duplicate authority, security bypass, or stuck request |
| H7 | Clean local staging reproduces the candidate | runbook deviations and gate result | undocumented source edit/manual recovery required |

## Fixed Workload

- Model: Qwen2.5-0.5B, exact revision and SHA-256 frozen in the campaign.
- Layout: three contiguous stages, exact stage artifacts and plan digest frozen.
- Prompts: checked-in non-sensitive corpus with token IDs and expected greedy
  outputs; no prompt selection after results.
- Input: 32, 128, and 512 token classes; acceptance performance uses one frozen
  representative class.
- Output: 32 greedy tokens, batch one.
- Security: normal permissions, NAC-ABE, tokens and replay checks in MiniNDN;
  production identities and cryptographic-strength evidence are deferred to
  Spec 106.

## Cells and Order

### E: Evidence Integrity

1. deterministic provider;
2. wiring-only provider;
3. real CPU ONNX providers;
4. declared CUDA requirement on the CPU-only host (must fail closed);
5. mixed CPU/synthetic or conflicting device evidence;
6. missing evidence;
7. artifact/plan digest mismatch.

These are deterministic contract cells. All invalid cells must BLOCK.

### C: Correctness

For every prompt and context class:

1. single-node full model;
2. local staged ONNX;
3. MiniNDN three-stage full-context;
4. MiniNDN cache-hit decode;
5. cache miss with full-context rebuild;
6. cache miss with delta-only failure.

Record tokens, logits/top-k diagnostics where available, cache bindings, stage
hashes and terminal reasons. No performance comparison uses cold export/load.

### P: MiniNDN Performance

- Warmup outside measurement.
- Three prespecified repetitions.
- 60-second measured window.
- Offered load: 1 RPS; request cap `ceil(rate * 60)`.
- Concurrency fixed by preflight and identical across single/distributed matched
  cells.
- INFO metrics only; TRACE disabled.
- No automatic replacement run.
- MiniNDN application permission/token/bootstrap evidence is retained, but its
  dummy-keychain environment is explicitly not cryptographic-strength evidence.

Primary: completion, achieved RPS, p50/p95/p99, single/distributed p95 ratio,
TTFT, inter-token latency. Secondary: stage compute/fetch/publish/queue, bytes,
host memory/process RSS, cache events. Higher-rate search is a later hypothesis;
it cannot be added after observing a pass.

### S: Scheduler Stress

Deterministic local stress with 1,000 unresolved futures, bounded queue, overflow,
deadline expiry, cancellation and shutdown. Sample process thread count and
memory before, during and after. This cell establishes a resource bound, not
network performance.

### F: Fault Matrix

| Fault | Injection point | Required result |
|---|---|---|
| provider kill | stage active | cancel epoch 0; epoch 1 or terminal no replacement |
| provider restart | after cache creation | new boot ID; old KV rejected |
| straggler | role compute | bounded deadline and one replacement |
| missing segment | dependency fetch | exact missing-dependency reason |
| hash mismatch | dependency/artifact | fail closed; no execution on bad bytes |
| stale telemetry | before lease commit | revalidate and reject/replan |
| cache eviction | decode | rebuild from full context or exact failure |
| late epoch-0 output | after epoch 1 | observed and ignored |

Five prespecified seeds/repetitions are used for stochastic loss/provider timing.
All failures are retained.

### O: Local MiniNDN Operations

1. install into two successive clean local staging directories;
2. doctor and application-security-path gate with MiniNDN evidence explicitly
   labeled non-production;
3. matched single/distributed canary;
4. scheduled provider restart;
5. release N -> N+1 upgrade;
6. N+1 -> N rollback;
7. 24-hour local MiniNDN soak at the frozen 1 request/second offered load;
8. emit `physicalProductionOverall=DEFERRED` and the Spec 106 handoff.

## Stop Rules

- stop before local operations if MiniNDN evidence, correctness, scheduler,
  application-security-path, or fault dimension BLOCKS;
- stop a run on wrong token, security bypass, corrupt authoritative Repo state,
  unbounded thread/memory growth, or host safety threshold;
- retain the run and diagnose; do not silently retry;
- capacity/service failure in tools preserves the last verified checkpoint and
  retries the tool operation only, never the measured campaign.

## Statistical and Evidence Rules

- Report each repetition and aggregate; never show best-run-only values.
- Completion intervals accompany small-sample rates.
- p50/p95/p99 are descriptive; no causal claim from one topology.
- Compare matched cells only: model, artifacts, security, topology, duration,
  warmup, backend, logging and request schedule must agree.
- TRACE, dummy keychain, deterministic runner and configured-only resource cells
  are explicitly non-comparable to production performance.
- Report negative or non-significant outcomes without relabeling them as gains.

## Planned Fallacy Scan (11/11)

1. ecological inference;
2. Simpson's paradox;
3. Berkson/collider selection;
4. base-rate neglect;
5. reverse causality;
6. regression to the mean;
7. survivorship bias;
8. look-elsewhere effect;
9. researcher degrees of freedom/forking paths;
10. correlation-to-causation;
11. overgeneralization from MiniNDN/small Qwen/three nodes.

Every final validation report states applicability or mitigation for all eleven.

## Canonical Evidence Layout

```text
results/spec105-<cell>-<unique-id>/
├── resolved-profile.json
├── command.txt
├── environment.json
├── execution-evidence.json
├── request-results.csv
├── stage-timings.csv
├── telemetry.csv
├── fault-events.jsonl
├── raw-logs/
└── release-gate.json
```
