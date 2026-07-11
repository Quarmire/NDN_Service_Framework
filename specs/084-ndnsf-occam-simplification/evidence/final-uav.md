# Final UAV Acceptance

Child 089's canonical campaign ran three matched MiniNDN trials at 5% one-way
loss. All 3/3 completed, seven chunks were recovered by FEC, maximum pending
bytes were 21,600, maximum frame gap was zero, mean run-level p50 was 53.5 ms,
and p95 was 120.0 ms.

Final integration smoke:

```bash
timeout 240s python3 Experiments/NDNSF_UAV_Stream_Parity_Campaign.py \
  --out results/spec084-final/uav-stream-loss5 \
  --runs 1 --auto-stop-seconds 8
```

The run completed 1/1 at 5% one-way loss, recovered one chunk with FEC, held
maximum pending bytes to 14,400, recorded zero frame gap and zero stale-session
acceptance, and measured p50 64 ms and p95 120 ms. C++ stream/UAV protocol
tests, mission and authority tests, static-transfer boundary checks, and all six
security regressions are recorded by child 089.

