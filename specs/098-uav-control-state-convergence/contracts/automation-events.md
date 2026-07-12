# Automation Event Contract

```text
UAV_AUTO_CONTROL_PHASE phase=<wait-begin|satisfied|expired|dispatch|terminal>
drone=<id> step=<arm|takeoff|land|emergency_stop|sequence>
prerequisite=<telemetry-ready|armed|airborne|disarmed|none>
timestamp_ms=<integer> elapsed_ms=<integer> reason=<code>
```

Rules:

- Every `wait-begin` ends at `satisfied`, `expired`, or shutdown terminal.
- Every command has at most one `dispatch` event per sequence.
- `expired` never causes an automatic command retry or safety bypass.
- Events contain no request payload, token, certificate, credential, key, or
  private telemetry value.
