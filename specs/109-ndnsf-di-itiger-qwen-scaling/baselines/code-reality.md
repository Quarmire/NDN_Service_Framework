# Spec 109 pre-change code reality

Captured against source snapshot `sha256:8dce660b3f8d568952650006fc026b9f5449b41add1cd9beea94a5960fc946cc`.
This is a descriptive baseline, not acceptance evidence.

## Model and stage binding

- `Experiments/NDNSF_DI_LlmPipeline_Minindn.py` defaults `--qwen-model` and
  `prepare_policy(... qwen_model=...)` to `Qwen/Qwen2.5-0.5B-Instruct`.
- `write_native_qwen_bundle()` emits `/Model/Qwen2.5-0.5B-Instruct` in both the
  execution plan and service manifest instead of deriving it from a sealed model
  registry entry.
- The same function creates dependencies with `for index in range(2)`, assumes
  a final stage at `index == 2`, and therefore hard-codes a three-stage shape.
- Stage artifacts are taken from the generated service manifest, but their NDN
  names are fixed to `/Artifact/QwenPilot/Stage/<index>` and are not bound to a
  model revision, tokenizer digest, export digest, or campaign identity.

## Backend binding

- Each emitted runner metadata record sets `executionProvider` to `cpu`,
  `allowCpuFallback` to `false`, and has no allocation-local CUDA device map.
- `OnnxRuntimeModelRunner.cpp` already supports requested `cpu` or `cuda`,
  normalizes the ONNX Runtime provider spelling, rejects unavailable CUDA when
  fallback is disabled, and records whether CPU fallback was used. Spec 109
  should parameterize and expose this existing boundary instead of introducing a
  second backend-selection mechanism.
- The current experiment acceptance/diagnostic branches explicitly require the
  `qwen-onnx-cpu-native` runtime in several Spec 107 paths. Those paths are not
  GPU acceptance authority.

## Runtime and security boundary retained

- `DI_NativeProviderExecutable.cpp` consumes an execution plan and service
  manifest, materializes runner specs, creates provider-local identities and
  permissions, registers the collaboration handler, and publishes readiness.
- Spec 109 may bind parameterized artifacts and backend evidence into those
  existing inputs. It must not change NDNSF wire names, NAC-ABE routing,
  UserToken/ProviderToken replay protection, or provider-permission ownership.

## Consequence for live work

The current code can describe a CPU 0.5B three-stage native candidate, but it
cannot establish a digest-bound, arbitrary-size, all-CUDA Qwen acceptance
candidate. In addition, the exact predecessor lock records Spec 107 T027 and
T028-T038 plus Spec 108 T091-T102 as incomplete. No Spec 109 candidate job is
eligible until those exact entries are materialized and pass.
