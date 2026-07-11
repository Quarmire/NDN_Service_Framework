# Diagnostic Event Contract

Generic Targeted event:

```text
GS_TARGETED_PHASE phase=<queued|dispatch-rejected|dispatched|response|timeout>
provider=<name> service=<name> request_id=<name-or-none>
timestamp_ms=<integer> elapsed_ms=<integer> status=<value>
```

UAV control event:

```text
UAV_CONTROL_COMMAND phase=<attempt|blocked|busy|response|timeout>
drone=<id> command=<name> timestamp_ms=<integer>
elapsed_ms=<integer> accepted=<true|false> reason=<code>
```

Events contain no request payload, tokens, certificates, credentials, or key
material.
