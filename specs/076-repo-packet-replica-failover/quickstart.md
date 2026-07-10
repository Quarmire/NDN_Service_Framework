# Quickstart: Repo Packet Replica Failover

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 tests/python/test_ndnsf_repo_exact_packets.py -v

sudo -n -E python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py \
  --exact-packet-failover-smoke \
  --nlsr-wait-s 10 \
  --repo-start-wait-s 15 \
  --output-dir results/distributed_repo_exact_packet_failover_minindn
```

Expected: Repo A exits after one successful packet; Repo B is called from the
first packet name and supplies the complete wire-identical set.

Accepted on 2026-07-10 with four packets. All eight result checks passed; total
latency was 50,772 ms and failover latency was 42,735 ms. Expected and actual
wire SHA-256 arrays were identical. The long timeout is
recorded as a performance follow-up rather than a correctness failure.
