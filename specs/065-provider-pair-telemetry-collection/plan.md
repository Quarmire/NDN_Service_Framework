# Implementation Plan: Provider-Pair Telemetry Collection

## Summary

Bridge existing dependency-edge ndnping evidence into the reusable core
`PeerNetworkMetric` and `ProviderNetworkMatrix` envelopes. This makes provider
pair facts visible in NativeTracer `summary.json` while keeping DI-specific
planning and dependency semantics in NDNSF-DI.

## Design

1. Add harness helpers that parse `dependency-edge-ndnping-rtt-stats.json`.
2. Convert each valid row to a `PeerNetworkMetric`.
   - `producerPrefix` -> `src_peer`
   - `consumerPrefix` -> `dst_peer`
   - `summaryMs.p50` or median `rttsMs` -> `rtt_ms`
   - `summaryMs.stddev` -> `jitter_ms`
   - `expectedBytes` -> `bytes_sampled`
3. Use conservative fallback bandwidth with reduced confidence because ndnping
   proves RTT but not throughput.
4. Add `providerPairTelemetry` to NativeTracer summary output in the final
   collection path.
5. Keep missing evidence non-fatal.
6. Add Python campaign regression coverage for fixture parsing and matrix
   consumption.

## Validation

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments \
  python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py

git diff --check
```

