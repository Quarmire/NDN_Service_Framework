# Negative ACK Early-Stop MiniNDN Evidence

Date: 2026-07-02

## Goal

Quantify whether known-provider negative ACK early-stop reduces user-visible
waiting time when every explicitly addressed provider rejects a request.

## Command

```bash
sudo -n python3 Experiments/NDNSF_NegativeAck_Minindn.py \
  --output-dir results/negative_ack_early_stop_minindn_20260702_195453 \
  --ack-timeout-ms 9000 \
  --user-timeout-s 35
```

The MiniNDN topology was `Experiments/Topology/AI_Lab.conf`.

Nodes:

- controller: `memphis`
- user: `memphis`
- provider A: `ucla`
- provider B: `wustl`

Both providers rejected `/HELLO` with negative ACKs:

- provider A: `QUEUE_FULL`
- provider B: `MODEL_UNAVAILABLE`

## Result

Summary file:

```text
results/negative_ack_early_stop_minindn_20260702_195453/negative-ack-early-stop-minindn-summary.json
```

| Case | Known providers | Early stop | Elapsed | Negative ACKs |
| --- | --- | --- | ---: | --- |
| `user-known-providers` | `A,B` | yes | 3174.77 ms | `QUEUE_FULL`, `MODEL_UNAVAILABLE` |
| `user-discovery-no-early-stop` | none | no | 23596.96 ms | `QUEUE_FULL`, `MODEL_UNAVAILABLE` |

The known-provider case returned about 7.43x faster than the discovery-style
case. It stopped before the 9000 ms ACK collection window because all known
providers had already returned negative ACKs. The discovery-style case recorded
the same two negative ACK reasons, but did not fail early because the user could
not prove that all possible providers had rejected the request.

## Interpretation

This validates the intended conservative semantics:

- negative ACKs are diagnostic for discovery-mode invocation;
- negative ACKs become terminal only when the request has an explicit complete
  provider set;
- early-stop avoids waiting for the full ACK/request timeout when the complete
  known provider set has already rejected the request.

An attempted NativeTracer provider-admission stress run also produced provider
side `PROVIDER_BUSY` decisions, but the user did not record those negative ACKs
and each request waited about 60.5 seconds. That run is not valid early-stop
evidence yet; it shows that the NativeTracer DI harness needs a follow-up if we
want provider-admission rejection to drive user-side early-stop in the DI path.
