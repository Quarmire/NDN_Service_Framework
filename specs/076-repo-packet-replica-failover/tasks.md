# Tasks: Repo Packet Replica Failover

## Phase 1 - Fetch routing contract

- [x] T001 [US1] Add a unit test proving `FETCH_PACKET_PREPARE.forwardingHints` reaches native exact fetch in `tests/python/test_ndnsf_repo_exact_packets.py`
- [x] T002 [US1] Pass validated forwarding hints through `NetworkDistributedRepoClient.fetch_packet` in `NDNSF-DistributedInference/ndnsf_distributed_inference/repo.py`

## Phase 2 - Deterministic client failure barrier

- [x] T003 [US1] Add secondary Repo, trigger, resume, and wait-time CLI options in `examples/python/NDNSF-DistributedRepo/generic_object_store/client.py`
- [x] T004 [US1] Seed the identical packet set to two explicit replicas in `examples/python/NDNSF-DistributedRepo/generic_object_store/client.py`
- [x] T005 [US1] Add experiment-only packet-call recording and first-primary-packet barrier in `examples/python/NDNSF-DistributedRepo/generic_object_store/client.py`
- [x] T006 [US2] Emit failover attempts, latency, exact-name, wire-hash, and whole-set restart checks in `examples/python/NDNSF-DistributedRepo/generic_object_store/client.py`

## Phase 3 - MiniNDN process failure

- [x] T007 [US1] Add `--exact-packet-failover-smoke` and clean both replica stores in `Experiments/NDNSF_DistributedRepo_Generic_Minindn.py`
- [x] T008 [US1] Start failover verification, wait for the trigger, terminate Repo A, and release the client in `Experiments/NDNSF_DistributedRepo_Generic_Minindn.py`
- [x] T009 [US2] Validate the failover result contract and preserve logs/results in `Experiments/NDNSF_DistributedRepo_Generic_Minindn.py`

## Phase 4 - Documentation and verification

- [x] T010 [P] [US2] Document exact packet replica failover in `NDNSF-DistributedRepo/README.md`
- [x] T011 [P] [US2] Synchronize failover documentation in `NDNSF-DistributedRepo/README_ch.md`
- [x] T012 Run exact packet, tiered-cache, and discovery Python regressions
- [x] T013 Run the two-replica MiniNDN failover acceptance and inspect JSON evidence
- [x] T014 Run `py_compile`, `git diff --check`, Spec convergence, and GSD health
