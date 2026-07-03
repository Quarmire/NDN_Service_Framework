# Quickstart: Runtime Doctor

Run the HELLO profile doctor:

```bash
python3 tools/ndnsf_runtime.py doctor \
  --profile examples/hello.runtime.json \
  --fix \
  --event-log /tmp/ndnsf-runtime-events.jsonl \
  --write-resolved /tmp/ndnsf-runtime-resolved.json
```

The doctor checks policy/trust files, build outputs, NFD socket status, and the
bootstrap token file. With `--fix`, a missing token file is generated from the
policy identities using 8-character tokens.

The event log is JSONL:

```json
{"event": "DOCTOR_START", "...": "..."}
{"event": "TOKEN_FILE_LOADED", "...": "..."}
{"event": "DOCTOR_RESULT", "ready": true}
```

Run the NativeTracer DI profile doctor:

```bash
python3 tools/ndnsf_runtime.py doctor \
  --profile examples/di-native-tracer.runtime.json \
  --event-log /tmp/ndnsf-di-runtime-events.jsonl \
  --write-resolved /tmp/ndnsf-di-runtime-resolved.json
```

This preflight checks the MiniNDN harness, topology, Qwen tiny proportional
model spec, provider profiles, required C++ smoke binaries, and expected
topology nodes. The resolved JSON includes the recommended MiniNDN command under
`profile.distributed_inference.native_tracer.command`, and the event log
includes `DI_NATIVE_TRACER_PREFLIGHT`.

Use the same profile to launch the NativeTracer harness:

```bash
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out /tmp/ndnsf-di-profile-run
```

Use `--runtime-resolved /tmp/ndnsf-di-runtime-resolved.json` when the launch
should consume the exact resolved paths produced by the doctor. Explicit command
line flags override profile defaults.
