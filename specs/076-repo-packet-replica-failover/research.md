# Research: Repo Packet Replica Failover

## Decision 1: Use forwarding hints, not renamed replica packet names

The Data name is application identity and stays unchanged. The Repo provider
identity returned by preparation disambiguates which replica should answer.

## Decision 2: Inject failure outside production code

Shared trigger/resume files coordinate the client and MiniNDN harness. This is
deterministic and does not add testing controls to the Repo wire protocol.

## Decision 3: Restart the complete set

Combining packet fragments from multiple attempts would complicate integrity,
ordering, and trust reasoning. The current local-result-per-replica algorithm is
kept; the experiment proves its behavior under real process failure.
