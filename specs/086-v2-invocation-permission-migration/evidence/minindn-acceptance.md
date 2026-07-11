# MiniNDN Normal And Targeted Acceptance

**Date**: 2026-07-11

Both modes used `Experiments/Topology/AI_Lab.conf`, with the controller and
user on `memphis`, one provider on `ucla`, 2 warmup requests, 10 measured
requests, a 500 ms closed-loop interval, a 200 ms ACK timeout, and a 3000 ms
service timeout. The Targeted command differed only by adding `--targeted`.

Common command prefix:

```bash
sudo -n -E python3 -B Experiments/NDNSF_NewAPI_Minindn_Perf.py \
  --topology-file Experiments/Topology/AI_Lab.conf \
  --controller-node memphis --user-node memphis \
  --provider-nodes ucla --providers 1 \
  --duration 35 --warmup 7 --interval-ms 500 --max-requests 10 \
  --strategy first-responding --ack-timeout-ms 200 --timeout-ms 3000 \
  --nlsr-converge-seconds 2 --controller-settle-seconds 2 \
  --provider-start-gap-seconds 0 --post-ready-settle-seconds 2 \
  --startup-settle-seconds 1 --provider-ready-timeout-seconds 30 \
  --nfd-log-level ERROR
```

| Mode | Result directory | Completion | p50 | p95 | Provider ACKs including warmup |
|---|---|---:|---:|---:|---:|
| Normal | `results/spec086-v2-normal-targeted/normal` | 10/10 | 59.432 ms | 95.507 ms | 12 |
| Targeted | `results/spec086-v2-normal-targeted/targeted` | 10/10 | 32.541 ms | 59.941 ms | 3 |

Both modes had zero timeout and zero pending calls at shutdown. Targeted's
three ACKs across twelve total calls show that the initial bootstrap/refill
used the normal exchange and later calls consumed cached one-time Targeted
token pairs without repeating ACK/Selection for every request.

The original entry baseline omitted MiniNDN latency. The parent commit was
subsequently measured with benchmark-only instrumentation; see
`pre-migration-minindn-baseline.md`. Normal p95 improved by 0.91 percent and
Targeted p95 regressed by only 0.76 percent, so both pass the 15 percent gate.
