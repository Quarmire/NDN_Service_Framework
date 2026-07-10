# Requirements Checklist: Online Replica Repair After Recovery

- [x] Goal distinguishes write quorum from desired replication.
- [x] Recovery state machine includes Repo and sidecar rejoin.
- [x] Existing durable repair path is reused instead of duplicated.
- [x] Exact packet names, wire bytes, and security checks remain invariant.
- [x] Measurement identifies outage-window objects explicitly.
- [x] Acceptance requires real MiniNDN repair to the recovered target.
- [x] Partial or negative results must be retained.
- [x] Slides and NDN-SVS are explicitly out of scope.
