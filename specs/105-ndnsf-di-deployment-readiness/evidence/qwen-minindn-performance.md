# Qwen MiniNDN Performance Gate

**Executed**: 2026-07-12  
**Tasks**: T047-T048  
**Verdict**: BLOCK  
**Stop rule**: triggered; no replacement run permitted

## Frozen profile

- Model: Qwen2.5-0.5B-Instruct, three contiguous ONNX stages
- Workload: prompt `NDNSF deployment pilot`, batch one, 32 greedy tokens
- Runtime: observed C++ ONNX Runtime 1.26.0 CPU providers
- Schedule: open loop, 1 generation request/second for 60 seconds
- Request cap: 60 per repetition
- Repetitions: exactly three, INFO logging
- Client concurrency: 4 asynchronous workers, fixed by preflight
- Provider workers: one per stage
- ACK timeout: 1,500 ms; request timeout: 120,000 ms
- Retries/replacements: none
- Security path: normal MiniNDN permissions, NAC-ABE routing and one-time tokens;
  dummy keychain means non-production cryptographic strength

The matched single-node identity and 32-token oracle are recorded in
`qwen-matched-baseline.md`. Its three measured local staged generations had
p50 6,787.63 ms, p95 6,854.20 ms, TTFT p50 177.58 ms, and inter-token p50
204.35 ms.

## Prespecified repetitions

| Run | Result directory | Offered | Complete at cutoff | Failed at cutoff | Unfinished | Completion | Successful token subrequests |
|---|---|---:|---:|---:|---:|---:|---:|
| 1 | `results/spec105-qwen-pilot-run1-20260712-0450` | 60 | 0 | 0 | 60 | 0% | 953 |
| 2 | `results/spec105-qwen-pilot-run2-20260712-0454` | 60 | 0 | 0 | 60 | 0% | 891 |
| 3 | `results/spec105-qwen-pilot-run3-20260712-0500` | 60 | 0 | 0 | 60 | 0% | 923 |

Run 1 and Run 2 used the original cleanup path, whose non-daemon async workers
kept Python alive after the measurement summary and caused the outer harness to
exit 1 at 210 seconds. Run 3 used non-wait shutdown plus hard process exit and
returned the intended gate exit 2. The cleanup correction occurred only after
the identical measurement cutoff; it did not change offered requests, client
or provider concurrency, ACK timeout, request timeout, tokens, or measured
completion. All three raw outcomes are retained, and no fourth run was made.

Run 3 shutdown callbacks subsequently labeled 56 outstanding requests failed;
their observed ages ranged from 126.012 to 185.013 seconds. They remain
unfinished at the fixed measurement cutoff and are not counted as completed.

For zero successes among 180 offered requests, the aggregate 95% Wilson upper
bound is approximately 2.1%. This is far below the required 99% completion.
Achieved complete-generation throughput is 0 RPS, below the required 0.95 RPS.

## Metric gate

| Required metric | Observed | Verdict |
|---|---|---|
| Completion | 0/60 in each run; 0/180 aggregate | BLOCK |
| Achieved throughput | 0 complete generations/s | BLOCK |
| Token equality | no complete campaign generation; T046 passed 32/32 independently | NOT ESTABLISHED IN CAMPAIGN |
| p50/p95/p99 | undefined with zero completed generations | BLOCK |
| Distributed/single-node p95 | undefined in campaign; T046 diagnostic was 25,320.02 / 6,854.20 = 3.69x | BLOCK |
| TTFT/inter-token | baseline available; no completed campaign generation distribution | BLOCK |
| >=99% stage decomposition | partial token-stage logs only; no completed request set | BLOCK |
| Resource metrics | not yet implemented by T049-T055 and absent here | BLOCK |
| Real CPU evidence | all providers reported `onnxruntime-cpu`, ORT 1.26.0 and exact digests | PASS |
| Physical production | outside MiniNDN-only scope | DEFERRED to Spec 106 |

The workload accumulated backlog rather than dropping the offered schedule.
Stage 0 executed ONNX only 28 times in each repetition while its exact-forward
memoization reused deterministic outputs across identical sessions; Stages 1
and 2 executed approximately once per successful token subrequest. Even with
this favorable Stage-0 memoization, no 32-token generation completed before the
hard cutoff. This result must not be presented as deployable 1 RPS capacity.

## 11/11 fallacy scan

| # | Fallacy | Applicability / mitigation |
|---:|---|---|
| 1 | Ecological inference | One local host and MiniNDN topology cannot establish physical fleet behavior; physical verdict remains DEFERRED. |
| 2 | Simpson's paradox | Each of the three repetitions is shown separately; the aggregate does not hide a passing subgroup. |
| 3 | Berkson/collider selection | No latency percentile is computed from only completed requests because there were none; partial token survivors are not selected as generations. |
| 4 | Base-rate neglect | Denominator is all 180 offered requests, including all unfinished requests. |
| 5 | Reverse causality | The report does not attribute failure solely to NDN; CPU service time, four-worker client concurrency, queues and protocol overhead coexist. |
| 6 | Regression to the mean | The first two failed outcomes are retained; no best-run replacement was performed. |
| 7 | Survivorship bias | Partial token subrequests and all unfinished generations are reported, not discarded. |
| 8 | Look-elsewhere effect | Rate, duration, token count and three repetitions were frozen before the campaign; no alternate rate search follows. |
| 9 | Researcher degrees of freedom | Cleanup changed only post-cutoff termination. Concurrency, timeouts, workload and acceptance thresholds stayed fixed. |
| 10 | Correlation to causation | Results falsify serviceability for this profile but do not isolate a single causal component. |
| 11 | Overgeneralization | Claims are limited to local CPU, Qwen2.5-0.5B, three stages and this topology; no physical or larger-model claim is made. |

## Decision

H3 and SC-002 are falsified for this immutable candidate. SC-003 and the requested
latency/resource evidence also fail or remain undefined. The original Phase 4
checkpoint stopped the unchanged campaign before T049. Increasing timeout,
reducing offered load, changing worker counts in place, or adding a fourth run to
this campaign remains prohibited.

Revision R1 now permits independent implementation and a separately identified,
preregistered campaign only after deterministic generation-scheduler validity
tasks T049-T051 pass. The new campaign cannot replace or be pooled with these
runs, and all original thresholds remain fixed. Spec 106 remains deferred and
must not be used to bypass this local algorithm/runtime gate.
