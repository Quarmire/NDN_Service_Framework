# Requirements Checklist: Bounded Parallel Replica Repair

- [x] Durable ordering fields and migration are specified.
- [x] Priority order is deterministic and testable.
- [x] `ServiceUser` control operations remain serialized.
- [x] Only independent data transfers are parallelized.
- [x] Concurrency is bounded and configurable.
- [x] Worker exceptions preserve durable retry behavior.
- [x] Matched baseline/treatment variables are fixed.
- [x] Negative campaign results must be retained.
- [x] Existing security and receipt invariants remain mandatory.
