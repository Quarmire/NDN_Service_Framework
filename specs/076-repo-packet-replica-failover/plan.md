# Implementation Plan: Repo Packet Replica Failover

**Spec**: `specs/076-repo-packet-replica-failover/spec.md`

## Summary

Use the Repo identity returned by `FETCH_PACKET_PREPARE` as a forwarding hint,
then extend the exact-packet example and MiniNDN harness with deterministic
mid-batch process termination. Production failover remains the atomic
replica-loop already implemented by Spec 075; only the experiment adds failure
coordination hooks.

## Technical Context

- Python `NetworkDistributedRepoClient`, native exact Data fetch helper.
- Existing `DistributedRepo.get_signed_packets` replica loop.
- MiniNDN process handles and shared result directory.
- Existing exact packet state and summary JSON.

## Constitution Check

- Dynamic NDNSF service APIs and security remain unchanged.
- No failure-control field enters production wire messages.
- CodeGraph was used before design.
- Spec Kit and GSD provide durable tasks and acceptance evidence.
- MiniNDN is the final network validation path.

## Design

### Targeted exact packet fetch

`fetch_packet` parses `forwardingHints` from `FETCH_PACKET_PREPARE` and forwards
them to `fetch_exact_data_packet`. The complete Data name remains unchanged.

### Deterministic failure barrier

The verification client wraps its local `fetch_packet` call only in failover
test mode. After the first successful primary packet, it writes a trigger JSON
file and waits for a resume file. The harness observes the trigger, terminates
Repo A, writes the resume file, and waits for verification.

### Atomic evidence

The wrapper records every `(repo, packetName, success)` call. Success requires:

- one successful primary packet before termination;
- a later primary failure;
- secondary calls equal the complete ordered `packetNames` list;
- final names and hashes equal the seed state;
- measured failover elapsed time is finite and recorded.

## Verification

1. Python unit test confirms forwarding hints reach native exact fetch.
2. Existing atomic failover unit test remains green.
3. Exact/tiered/discovery Repo regressions remain green.
4. New MiniNDN failover mode stores two replicas and kills Repo A mid-batch.
