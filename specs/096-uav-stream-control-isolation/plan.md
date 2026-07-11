# Implementation Plan

## Constitution Check

- Use the canonical V2/Targeted UAV runtime unchanged.
- Preserve NAC-ABE, permissions, tokens, and replay checks.
- Use CodeGraph before source edits and MiniNDN for final evidence.
- Keep all policy and parsing in the experiment layer.

## Design

1. Extend the existing campaign command builder with an `include_video` switch
   that defaults true, preserving Spec 095 callers.
2. Extend the existing parser with the same switch. When video is absent,
   video completion and structured stream metrics are not required; control
   markers and process exit remain mandatory.
3. Add a thin isolation campaign that imports the canonical Spec 095 campaign
   helpers rather than duplicating topology, parsing, acceptance, or CSV code.
4. Freeze five workload cells and three repetitions at 5% one-way loss.
5. Write per-run JSON/CSV and per-cell CSV. Never retry or replace a run.
6. Compare video-only with combined within parity, and control-only with the
   control component of combined cells. Report counts and uncertainty only.

## Primary Matrix

| Cell | Video | Control | Parity | Runs | Video duration |
|---|---:|---:|---:|---:|---:|
| control-only | no | yes | n/a | 3 | n/a |
| video-only-fec0 | yes | no | 0 | 3 | 60 s |
| video-only-fec1 | yes | no | 1 | 3 | 60 s |
| combined-fec0 | yes | yes | 0 | 3 | 60 s |
| combined-fec1 | yes | yes | 1 | 3 | 60 s |

## Validation

- Strict Spec Kit pre/post audit and convergence.
- Focused campaign unit tests and full Python suite.
- 15-run MiniNDN campaign, no retries.
- Machine-readable and human-readable evidence review.
- GSD health, CodeGraph sync, proposal-path scan, clean git.

## Rollback

Remove the thin isolation campaign and revert the two default-preserving helper
parameters. No runtime or wire migration is involved.
