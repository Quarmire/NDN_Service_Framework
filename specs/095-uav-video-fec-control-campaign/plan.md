# Implementation Plan

## Constitution Check

- Canonical V2 runtime and Targeted control calls remain unchanged.
- NAC-ABE, permission, token, and replay checks remain in the active path.
- CodeGraph caller inspection precedes source edits.
- Spec Kit owns requirements/tasks and GSD owns resumable campaign state.
- MiniNDN with 60-second measured windows is the final network gate.

## Boundaries

- Core: unchanged generic stream metadata/state.
- UAV Drone: validate and apply the requested XOR parity count.
- UAV Ground Station: expose one FEC request setting and report accepted state.
- Experiment: own topology matrix, repetitions, acceptance gates, summaries,
  and statistical description.

## Design

1. Add a `videoFecParityShards` constructor/config value to Ground Station and
   pass `fec_parity_shards` in `startVideoAttempt`.
2. Parse/clamp the field in `VideoPublisher::start`; zero data-only and one XOR
   parity use the existing publication loop.
3. Extend `NDNSF_UAV_GUI_Minindn.py` with the option and concurrent-video/control
   validation markers.
4. Replace the single-treatment campaign loop with deterministic cells:
   `loss x parity x repetition`. Write one topology file per loss under output.
5. Parse final decoded-frame counters and control markers from logs. Preserve
   each failed run; never auto-retry.
6. Aggregate by treatment and produce `campaign-summary.json`, per-run CSV, and
   treatment CSV. Interpret FEC recovery and delivery gaps, not just process RC.

## Primary Matrix

| Loss | Parity | Runs | Duration | Purpose |
|---:|---:|---:|---:|---|
| 0% | 0 | 3 | 60 s | no-loss overhead/control |
| 0% | 1 | 3 | 60 s | no-loss parity overhead |
| 5% | 0 | 3 | 60 s | lossy control |
| 5% | 1 | 3 | 60 s | lossy FEC treatment |

An optional one-run 15% off/on pair is boundary evidence only.

## Validation

- Focused C++ protocol/UAV tests and Python campaign tests.
- Affected UAV build targets.
- Campaign dry-run matrix inspection.
- Primary 12-run MiniNDN campaign with 60-second windows.
- Strict Spec Kit audit, convergence, GSD health, CodeGraph sync, clean git.

## Rollback

The new setting defaults to one parity shard, preserving current runtime
behavior. Reverting the focused implementation commit removes the experiment
control without changing Core wire semantics.
