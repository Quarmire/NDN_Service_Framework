# Plan: DI Auto Full-Network Campaign

## Approach

Reuse the existing NativeTracer MiniNDN harness and the campaign utilities from
`run_layout_campaign.py`.

The new helper runs only the `auto` assignment across workload points:

```text
c1: requests=1, concurrency=1
c2: requests=2, concurrency=2
c4: requests=4, concurrency=4
```

Each run goes through the real full-network path:

```text
planner probe -> resolved assignment -> controller/provider bootstrap ->
provider --serve -> NativeTracer user driver -> dependency exchange
```

## Validation

1. Python syntax validation.
2. One-run campaign smoke for c1/c2/c4.
3. Repeated MiniNDN campaign for c1/c2/c4.
4. Check generated JSON and CSV summaries.

All validation steps are complete.

## Expected Result

```text
c1: selectedCandidate=single-provider-serial
c2: selectedCandidate=shared-backbone-current
c4: selectedCandidate=shared-backbone-current
```

The exact latency values are measured evidence. The important acceptance signal
is that all runs execute and the selected candidate matches the planner boundary
from Feature 017.

## Observed Result

The 5-run MiniNDN campaign completed 15 full-network runs:

```text
c1: 5/5 runs, 5/5 requests, auto -> single-provider -> single-provider-serial
c2: 5/5 runs, 10/10 requests, auto -> default -> shared-backbone-current
c4: 5/5 runs, 20/20 requests, auto -> default -> shared-backbone-current
```

Per-request workload latency evidence:

```text
c1 workloadMeanMeanMs=511.529, workloadP95MeanMs=511.529
c2 workloadMeanMeanMs=469.389, workloadP95MeanMs=502.756
c4 workloadMeanMeanMs=449.850, workloadP95MeanMs=487.826
```

The measured boundary is now executable, not only estimated: single request uses
the single-provider serial layout, while concurrent workloads use the
shared-backbone layout.

## Follow-Up Queue-Pressure Evidence

A later campaign reused the same auto helper after provider ready-queue timing
and planner queue-pressure fields were added:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_auto_assignment_campaign.py \
  --runs 2 \
  --out-root /tmp/ndnsf-di-024-auto-queue-campaign-2 \
  --provider-check-timeout 60 \
  --role-execution-delay-ms 75 \
  --workloads c1:1:1,c4:8:4,c8:16:8
```

Observed result:

```text
c1: 2/2 runs, 2/2 requests, auto -> single-provider -> single-provider-serial
c4: 2/2 runs, 16/16 requests, auto -> default -> shared-backbone-current
c8: 2/2 runs, 32/32 requests, auto -> default -> shared-backbone-current
```

The selected layout's planner queue pressure was `0.000 ms` for all workloads.
For c4/c8, the estimated single-provider queue pressure was `43.664 ms` and
`50.941 ms`, while the measured provider queue mean for the selected
shared-backbone runs stayed low at `0.558 ms` and `0.086 ms`.
