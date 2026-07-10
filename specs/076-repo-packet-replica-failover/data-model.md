# Data Model: Repo Packet Replica Failover

## FailoverAttempt

- `repoNode`
- `packetName`
- `success`
- `error`
- `timestampMs`

## FailoverBarrier

- Trigger file: primary Repo and first successful packet.
- Resume file: harness confirms the primary process has exited.
- Both are experiment-only artifacts.

## FailoverResult

- Seed manifest and expected packet hashes.
- Ordered attempt records.
- Primary successful count and secondary requested names.
- Total and failover latency.
- Boolean acceptance checks.
