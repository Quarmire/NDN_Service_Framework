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
