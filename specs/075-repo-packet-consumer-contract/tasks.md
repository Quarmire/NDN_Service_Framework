# Tasks: Repo Packet Consumer Contract

## Phase 1 - Contract and boundary tests

- [x] T001 [US1] Add C++ failing tests for ordered packet-set retrieval and exact complete-name validation in `NDNSF-DistributedRepo/examples/DistributedRepoExactPacketTest.cpp`
- [x] T002 [P] [US1] Add Python failing tests for ordered retrieval, missing packets, duplicates, and name mismatch in `tests/python/test_ndnsf_repo_exact_packets.py`
- [x] T003 [P] [US2] Add compatibility tests for opaque objects and the packet-backed payload view in `tests/python/test_ndnsf_repo_exact_packets.py`

## Phase 2 - C++ packet consumer

- [x] T004 [US1] Add `RepoClient::getDataPackets` contract in `NDNSF-DistributedRepo/include/ndnsf-distributed-repo/RepoClient.hpp`
- [x] T005 [US1] Implement count, uniqueness, decode, exact-name, and atomic retrieval checks in `NDNSF-DistributedRepo/src/RepoClient.cpp`
- [x] T006 [US1] Complete C++ positive and negative exact packet tests in `NDNSF-DistributedRepo/examples/DistributedRepoExactPacketTest.cpp`

## Phase 3 - Python packet consumer and guard

- [x] T007 [US1] Implement `NetworkDistributedRepoClient.fetch_signed_packets` in `NDNSF-DistributedInference/ndnsf_distributed_inference/repo.py`
- [x] T008 [US1] Add `DistributedRepo.get_signed_packets` facade and manifest resolution in `NDNSF-DistributedInference/ndnsf_distributed_inference/repo.py`
- [x] T009 [US2] Preserve and document `fetch_object` as a non-mutating payload view for packet-backed manifests in `NDNSF-DistributedInference/ndnsf_distributed_inference/repo.py`
- [x] T010 [US1] Complete Python packet consumer and boundary tests in `tests/python/test_ndnsf_repo_exact_packets.py`

## Phase 4 - Consumer migration and documentation

- [x] T011 [US3] Replace hand-written packet fetch loops with `get_signed_packets` in `examples/python/NDNSF-DistributedRepo/generic_object_store/client.py`
- [x] T012 [US3] Update the exact-packet MiniNDN acceptance path to prove the high-level consumer contract in `Experiments/NDNSF_DistributedRepo_Generic_Minindn.py`
- [x] T013 [P] [US3] Document the Repo/UAV/DI packet-versus-object decision table in `NDNSF-DistributedRepo/README.md`
- [x] T014 [P] [US3] Synchronize the decision table in `NDNSF-DistributedRepo/README_ch.md`

## Phase 5 - Verification and convergence

- [x] T015 Run the C++ exact packet, smoke, and tiered-cache tests
- [x] T016 Run Python exact packet, tiered-cache, and Repo discovery regressions
- [x] T017 Run MiniNDN exact-packet acceptance and verify the machine-readable summary
- [x] T018 Run `git diff --check`, Spec Kit analysis/convergence, and GSD health validation

## Dependencies

- T001-T003 establish the failing contract.
- T004-T006 implement the C++ path.
- T007-T010 implement the Python path and boundary guard.
- T011-T014 depend on the Python public API.
- T015-T018 verify the complete feature.

## Independent Acceptance

- **US1**: One high-level call returns every exact packet in manifest order or
  fails without a partial result.
- **US2**: Opaque objects still work; packet-backed manifests expose exact-wire
  and reassembled-payload read views without changing stored representation.
- **US3**: The canonical example and MiniNDN acceptance use the shared consumer
  API rather than manual per-packet loops.
