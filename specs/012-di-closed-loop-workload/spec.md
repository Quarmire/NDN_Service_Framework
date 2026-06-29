# Feature 012: DI Closed-Loop Workload Campaign

Status: Accepted

## Goal

Extend the NativeTracer user driver and MiniNDN campaign harness from one
request to a small closed-loop workload. This measures makespan, p50/p95
latency, and throughput under the same smallest Qwen artifacts and optional
provider-capacity pressure.

## Scope

- Preserve the existing single-request default behavior.
- Add a `--requests` parameter to the NativeTracer user driver.
- Run requests sequentially in one user process with the same ServiceUser,
  scope-key setup, and NDNSF collaboration path.
- Emit one JSON result line per request plus a workload summary line.
- Thread request count through the MiniNDN harness and campaign runner.
- Record campaign results for `default` and `single-provider` layouts.

## Non-Goals

- True concurrent outstanding collaboration requests.
- New C++ API or pybind async collaboration binding.
- Changing provider scheduling or NDNSF wire protocol.
- Changing the smallest Qwen NativeTracer model artifacts.

## Acceptance

- [x] `user_driver.py --requests N` emits N per-request results and a workload
  summary with makespan, p50, p95, and throughput.
- [x] Existing single-request parsing remains compatible with current harness.
- [x] MiniNDN harness records workload metrics in `summary.json`.
- [x] Campaign runner records request count and can compare layouts.
- [x] At least one full-network closed-loop smoke completes.
- [x] Results and next recommendation are recorded here.

## Accepted Results

The user driver now supports a closed-loop workload:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment default \
  --role-execution-delay-ms 75 \
  --requests 3 \
  --out /tmp/ndnsf-di-closed-loop-default-smoke \
  --provider-check-timeout 60
```

The driver emits one `NDNSF_DI_NATIVE_TRACER_USER_REQUEST` line per request and
an aggregate `NDNSF_DI_NATIVE_TRACER_USER_WORKLOAD` /
`NDNSF_DI_NATIVE_TRACER_USER_EXECUTION` line. The smoke produced three
successful requests with aggregate makespan `1194.105 ms`, p95 `522.855 ms`,
and throughput `2.512 rps`.

Small campaign:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 2 \
  --out-root /tmp/ndnsf-di-closed-loop-campaign-3req-75 \
  --provider-check-timeout 60 \
  --role-execution-delay-ms-list 75 \
  --requests 3
```

| Assignment | Runtime candidate | Runs | Requests/run | Makespan mean ms | Makespan p95 ms | Workload p95 mean ms | Throughput mean rps |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| `default` | `shared-backbone-current` | 2 | 3 | 1128.025 | 1137.698 | 480.911 | 2.660 |
| `single-provider` | `single-provider-serial` | 2 | 3 | 1231.884 | 1260.203 | 489.938 | 2.437 |

The shared-backbone layout reduced closed-loop makespan by `103.859 ms` mean and
improved throughput from `2.437 rps` to `2.660 rps` under the same 75 ms per-role
capacity pressure.

## Interpretation

This is still a closed-loop sequential workload, not true concurrent
outstanding requests. It strengthens the single-request capacity result because
the advantage persists across repeated requests in one user session. The next
step should add an async collaboration binding or C++ NativeTracer user driver
so multiple requests can be outstanding at once and provider worker queues can
be measured directly.
