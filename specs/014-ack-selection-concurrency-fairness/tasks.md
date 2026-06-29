# Tasks: ACK Selection Concurrency Fairness

- [x] F001 Trace ACK matching and duplicate/replay keys in `ServiceUser`.
- [x] F002 Trace collaboration selection scheduling and no-selection timeout.
- [x] F003 Implement request-scoped ACK/selection fairness fix.
- [x] F004 Build relevant C++/Python targets.
- [x] F005 Run sequential NativeTracer sanity.
- [x] F006 Run `requests=2`, `concurrency=2` concurrent diagnostic.
- [x] F007 Update docs with result and next step.

## Result

The concurrency failure was caused by provider-side hybrid key reuse across
different requester identities. Provider ACKs for the first worker attached a
wrapped message key. ACKs for the second worker reused the same provider send
key but did not attach a wrapped key, because the key had already been marked as
wrapped. The second worker runs with a distinct requester identity and process,
so it cannot rely on the first worker's cached provider key. It received ACK
Data but could not decrypt it, collected zero ACK candidates, and eventually
timed out before selection.

The fix makes provider ACK and RESPONSE messages request-scoped receiver
messages: the provider attaches a wrapped message key for each ACK/RESPONSE
publication and does not mark that key as globally wrapped for these
request-scoped messages. This preserves cached-key behavior for non-request
messages while making concurrent requester workers independent.

The user driver also now writes each child worker's full stdout/stderr to
`logs/user-worker-<index>.log`, which makes concurrent diagnostics easier to
audit.

## Validation Commands

Sequential sanity:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment default \
  --role-execution-delay-ms 75 \
  --requests 1 \
  --concurrency 1 \
  --out /tmp/ndnsf-di-014-sync-after-keyfix \
  --provider-check-timeout 60
```

Result: `status=SUCCESS`, `runnerMode=qwen-onnx-native`,
`securityBootstrap=executed`, `userExecution=executed`,
`dependencyExecution=executed`.

Concurrent diagnostic:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --core-trace \
  --assignment default \
  --role-execution-delay-ms 75 \
  --requests 2 \
  --concurrency 2 \
  --out /tmp/ndnsf-di-014-c2-provider-rebuilt \
  --provider-check-timeout 60
```

Result: `status=SUCCESS`, `requestCount=2`, `concurrency=2`,
`successCount=2`, `failureCount=0`, `meanMs=492.59`, `p95Ms=522.13`.
Provider logs show `NDNSF_DI_PROVIDER_HANDLER_TIMING event=end` for both
request sessions on Backbone, Head0, Head1, and Merge.
