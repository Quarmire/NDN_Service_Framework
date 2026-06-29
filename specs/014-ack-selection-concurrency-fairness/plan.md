# Plan: ACK Selection Concurrency Fairness

## Final Diagnosis

The NativeTracer concurrency failure was caused by request-scoped receiver key
distribution, not by provider execution and not by a request ID collision.
Provider ACKs reached the second worker before decryption, but the worker never
reported ACK decryption success. The provider had already marked its hybrid send
key as wrapped after ACKs for the first worker, so ACKs to the second worker did
not carry a wrapped message key. Because each concurrent worker has a distinct
requester identity and process-local key cache, the second worker could not
decrypt those ACKs, collected no candidates, and did not publish selection.

## Implemented Fix

Provider `ACK` and `RESPONSE` publications are now treated as request-scoped
receiver messages in `ServiceProvider::publishHybridEncodedMessage`. The
provider attaches a wrapped message key for each ACK/RESPONSE publication and
does not mark that send key as globally wrapped for those message types. This
lets separate requester identities decrypt their own ACK/RESPONSE messages
without depending on another worker's key cache.

`ServiceUser::makeRequestId` also now adds a random suffix to the timestamp
component so concurrent workers never share the same request ID string when
they start in the same timestamp granularity window. This was validated as a
good safety improvement, although it was not the final root cause.

The NativeTracer user driver now stores each child process log as
`logs/user-worker-<index>.log` so future concurrent request failures have direct
per-worker evidence.

## Steps

1. Trace `ServiceUser` ACK matching, duplicate detection, ACK collection timer,
   and collaboration selection scheduling. Complete.
2. Add focused lifecycle logs or counters if the state transition is ambiguous.
   Complete through per-worker logs and core trace.
3. Fix request-scoped ACK/RESPONSE key wrapping. Complete.
4. Rebuild `ndn-service-framework`, `di-native-provider`, and the Python
   wrapper. Complete.
5. Run sequential sanity. Complete:
   `/tmp/ndnsf-di-014-sync-after-keyfix`, `SUCCESS`.
6. Run concurrent diagnostic with core trace. Complete:
   `/tmp/ndnsf-di-014-c2-provider-rebuilt`, `SUCCESS`, 2/2 requests executed.
7. Update docs/spec with the measured result. Complete.

## Verification Notes

The concurrent success run used:

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

Observed summary: `status=SUCCESS`, `requestCount=2`, `concurrency=2`,
`successCount=2`, `failureCount=0`, `meanMs=492.59`, `p95Ms=522.13`.
Provider logs show handler end timing for both request sessions on Backbone,
Head0, Head1, and Merge.
