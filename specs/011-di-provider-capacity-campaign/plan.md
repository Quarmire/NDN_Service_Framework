# Plan: DI Provider Capacity Campaign

## Approach

Use a deterministic sleep inside the NativeTracer runner as a controlled capacity
pressure knob. This keeps the same smallest Qwen artifacts and avoids pretending
that artificial activation padding is real model work.

The delay is encoded in service-manifest artifact metadata as
`executionDelayMs`. The C++ ONNX runner sleeps after model inference and before
returning outputs, so provider timing records the pressure as role `executeMs`.
The fake manifest smoke runner uses the same metadata for local validation.

## Steps

1. Add `executionDelayMs` metadata support to the C++ ONNX runner and fake smoke
   runner.
2. Add `--role-execution-delay-ms` to `plan_tracer.py`; patch all NativeTracer
   role artifacts with that metadata and sidecar hashes.
3. Add `--role-execution-delay-ms` to the MiniNDN harness and record it in
   `summary.json` and optimization evidence.
4. Add `--role-execution-delay-ms-list` to `run_layout_campaign.py`; combine it
   with existing activation padding support.
5. Validate with syntax/build checks, local execution smoke, full-network smoke,
   and a small repeated capacity campaign.

## Expected Interpretation

If both layouts run the same per-role delay, single-provider may still be close
for a single request because roles run once either way. The campaign is still
useful because it records the clean experimental control. If the shared layout
only wins with concurrent requests or provider queue pressure, the next feature
should extend the user driver to issue multiple requests and measure makespan or
throughput under per-provider worker limits.
