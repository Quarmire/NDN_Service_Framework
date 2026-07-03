# Feature Specification: Runtime Doctor and Usability Harness

**Feature Branch**: `046-runtime-doctor-usability`

**Status**: Implemented

## Goal

Reduce NDNSF setup and regression friction by giving developers one profile,
one doctor command, one structured event stream, and shared regression helpers.

## Requirements

- **FR-001**: Provide a stdlib-only runtime doctor CLI that can run in fresh VMs
  and MiniNDN nodes.
- **FR-002**: The doctor MUST load a JSON runtime profile and write a resolved
  absolute-path configuration.
- **FR-003**: The doctor MUST generate a missing bootstrap token file from the
  configured policy identities when `--fix` is used.
- **FR-004**: Generated bootstrap tokens MUST be 8 characters.
- **FR-005**: The doctor MUST emit JSONL structured events for start, token-file
  load/generation, and final readiness.
- **FR-006**: Regression scripts SHOULD use shared helpers for NFD startup,
  log waiting, cleanup, and log tailing.
- **FR-007**: The doctor SHOULD support a DI NativeTracer profile that checks
  harness files, Qwen tiny proportional artifacts, provider profiles, required
  smoke binaries, expected topology nodes, and records a reproducible MiniNDN
  command.
- **FR-008**: The NativeTracer MiniNDN harness SHOULD accept the same runtime
  profile or resolved doctor JSON as defaults, while keeping explicit command
  line flags as overrides.
- **FR-009**: LLM NativeTracer campaign runners SHOULD accept the same runtime
  profile or resolved doctor JSON so single runs, doctor preflights, and RPS
  campaigns share configuration defaults.
- **FR-010**: Planner-only NativeTracer sweep helpers SHOULD accept the same
  runtime profile or resolved doctor JSON for output roots, Qwen model/provider
  artifacts, RPS defaults, and workload sizing defaults.

## Success Criteria

- `python3 tests/python/test_ndnsf_runtime_doctor.py` passes.
- `python3 tools/ndnsf_runtime.py doctor --profile examples/hello.runtime.json --fix`
  succeeds after a normal build.
- `python3 tools/ndnsf_runtime.py doctor --profile examples/di-native-tracer.runtime.json`
  succeeds after a normal build and emits `DI_NATIVE_TRACER_PREFLIGHT`.
- `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile
  examples/di-native-tracer.runtime.json --out <tmp>` succeeds in
  local-execution-only mode from profile defaults.
- `python3 tests/python/test_ndnsf_llm_campaign_runtime_profile.py` passes,
  proving the LLM campaign runner consumes runtime profile defaults.
- `python3 tests/python/test_ndnsf_sweep_runtime_profiles.py` passes, proving
  planner-only sweep helpers consume runtime profile and resolved doctor
  defaults.
- The token certificate bootstrap regression still passes.
- The aggregate security regression still passes.
