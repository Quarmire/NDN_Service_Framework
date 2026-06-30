# Feature 032: LLM Capacity Pressure Campaign

## Goal

Add a controlled compute-occupancy knob to deterministic LLM full-network
execution so greedy and proportional layouts can be compared under provider
capacity pressure. The previous campaign measured protocol overhead correctly,
but the deterministic runner returned immediately, so it did not stress the
single 8GB provider used by the greedy layout.

## Design

- Make the C++ deterministic native provider runner honor artifact metadata
  `executionDelayMs`.
- Let the LLM bundle generator override per-stage delay with
  `--stage-execution-delay-ms`.
- Thread the existing harness `--role-execution-delay-ms` value into the LLM
  bundle generator.
- Add the same delay option to the LLM full-network campaign runner.

## Validation

- Rebuild `di-native-provider`.
- Run a greedy/proportional full-network campaign with non-zero stage delay and
  concurrency greater than one.
- Record whether proportional remains an overhead-only layout or starts to
  reduce tail latency/throughput pressure.

## Interpretation

This remains a synthetic workload. A win here would show that proportional
layout helps when provider compute time is the bottleneck; it would not yet
prove real ONNX LLM performance.
