# 8 RPS Post-Processing Diagnosis

The first 8 RPS treatment completed its user workload, but the MiniNDN harness
aborted while collecting provider ACK runtime hints:

```text
AckMetadataDecodeError: typed provider capability envelope is required
```

Raw user execution reports 480/480 success, 8.0 offered RPS, 15.920 ms maximum
schedule slip, zero local backpressure, p50 198.1 ms, and p95 249.2 ms.

The failing provider log contains one character-interleaved observability
record. Line 3940 is only:

```text
NDNSF_DI_NATIVE_PROVIDER_ACK_DECISION provider=
```

The remainder appears around a concurrently written capacity line and at line
3944. This is a provider-process logging race, not evidence of a malformed wire
ACK. The collector must not reconstruct the line or silently count it as valid;
it must skip it, increment a parse-error counter, preserve a bounded diagnostic
sample, and continue collecting valid events.

The RED replay test reproduced the original exception. After the bounded
collector fix, the same raw log produces:

```text
valid ACK hint events: 2867
parse errors: 1
providers retained: llm-2gb, llm-4gb, llm-8gb
```

The focused runtime-aware campaign tests pass 31/31. The complete Python suite
passes 343 tests with one existing environment-dependent skip.
