# Feature 011: DI Provider Capacity Campaign

Status: Accepted

## Goal

Keep the same smallest Qwen NativeTracer ONNX artifacts and add a controlled
provider-capacity pressure experiment. The purpose is to test whether the
multi-provider shared-backbone layout becomes useful when one provider must run
multiple roles serially under extra per-role work, while distributed providers
can run roles on separate nodes.

## Scope

- Do not replace or enlarge the Qwen-derived ONNX artifacts.
- Add a per-role execution-delay metadata knob that simulates provider capacity
  pressure and is included in measured role `executeMs`.
- Thread the knob through NativeTracer policy generation, MiniNDN harness, and
  repeated campaign runner.
- Run a smoke campaign comparing `default` and `single-provider` layouts under
  at least two delay values.
- Record measured p50/p95/stddev and the interpretation for docs/slides/paper.

## Non-Goals

- Modeling GPU scheduling precisely.
- Changing the service invocation protocol.
- Adding new NDNSF wire messages.
- Downloading or generating a larger Qwen model.

## Acceptance

- [x] ONNX NativeTracer runners can apply optional per-role execution delay from
  artifact metadata.
- [x] Local fake runner smoke honors the same metadata so local execution checks
  remain meaningful.
- [x] `plan_tracer.py` can patch per-role delay metadata without changing model
  artifacts.
- [x] MiniNDN harness accepts and records a role delay parameter.
- [x] Campaign runner can sweep role delay values.
- [x] At least one full-network smoke campaign completes with all runs executed.
- [x] Results and next recommendation are recorded here.

## Accepted Results

The capacity campaign keeps the same smallest Qwen NativeTracer ONNX artifacts
and adds `executionDelayMs` metadata to every role artifact. The delay is
applied inside the C++ ONNX runner after model inference and before returning
outputs, so role timing records it as execution time rather than network
transfer.

Local smoke:

```bash
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --local-execution-only \
  --assignment default \
  --role-execution-delay-ms 25 \
  --out /tmp/ndnsf-di-capacity-local-smoke
```

The local timing CSV showed role `executeMs` values above 25 ms, confirming that
the metadata controls actual runner time.

Single-run delay sweep:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 1 \
  --out-root /tmp/ndnsf-di-capacity-campaign-smoke \
  --provider-check-timeout 60 \
  --role-execution-delay-ms-list 0,25

python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 1 \
  --out-root /tmp/ndnsf-di-capacity-campaign-smoke-50-75 \
  --provider-check-timeout 60 \
  --role-execution-delay-ms-list 50,75
```

| Role delay ms | Shared-backbone ms | Single-provider ms | Single minus shared ms |
| ---: | ---: | ---: | ---: |
| 0 | 285.664 | 179.470 | -106.194 |
| 25 | 345.130 | 318.482 | -26.648 |
| 50 | 419.097 | 388.562 | -30.535 |
| 75 | 469.112 | 473.932 | 4.820 |

The single-run sweep suggests a crossing near 75 ms per role. A 3-run campaign
at 75 ms confirmed the direction:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 3 \
  --out-root /tmp/ndnsf-di-capacity-campaign-75x3 \
  --provider-check-timeout 60 \
  --role-execution-delay-ms-list 75
```

| Layout | Runs | Mean ms | Stddev ms | p50 ms | p95 ms |
| --- | ---: | ---: | ---: | ---: | ---: |
| `shared-backbone-current` | 3 | 494.673 | 33.628 | 488.016 | 531.132 |
| `single-provider-serial` | 3 | 512.909 | 31.059 | 506.711 | 546.599 |

At 75 ms per role, shared-backbone was faster by 18.236 ms mean and 18.695 ms
p50. This is the first measured point where the current distributed layout beats
single-provider under controlled provider-capacity pressure.

## Interpretation

The previous activation-padding campaign showed that making intermediate data
larger only stresses NDNSF dependency exchange. This capacity campaign shows a
more useful boundary: when the work per role is high enough and one provider has
to execute all roles serially, distributing roles across providers can overcome
the dependency exchange overhead.

Next step: extend the user driver from one request to a small concurrent or
closed-loop workload and measure makespan, p95 latency, and throughput under
per-provider worker limits. That will connect provider capacity pressure to the
real DI goal more directly than single-request latency alone.
