# Preflight

Pending execution. Stable anchor:

```text
Spec 092 threaded rep1-rep3
commit: dbb880c
180/180 requests
mean throughput: 1.013324 RPS
worst maximum slip: 14.813 ms
720/720 dependency events
```

Execution commit is `2888e07`; the changes after runtime commit `dbb880c` are
Spec Kit/workflow documents only. The source worktree was clean before the
first treatment.

Dry-run expansion confirmed:

| Rate | Request cap | Driver | Duration | Concurrency |
|---:|---:|---|---:|---:|
| 2 RPS | 120 | threaded | 60 s | 4 |
| 4 RPS | 240 | threaded | 60 s | 4 |
| 8 RPS | 480 | threaded | 60 s | 4 |

All dry runs resolve to Qwen ONNX NativeTracer, AI_Lab, `llm-proportional`, the
same two-user workload fixture, one runtime-aware replan allowance, and no
provider-pair telemetry probe. Only target rate, request cap, output paths, and
the values derived from them differ.

Pre-run resources:

```text
available memory: 9.2 GiB
free disk: 8.3 GiB
stale MiniNDN/NFD/user/provider processes: none
minimum disk stop: 3 GiB
```
