# Implementation Plan

## Constitution Check

- Canonical V2 Targeted invocation and unified service names remain unchanged.
- NAC-ABE, permission, token, replay, and provider checks stay in the path.
- CodeGraph established callers and thread ownership before edits.
- MiniNDN is the final lifecycle/network verification path.
- Failed commands and runs remain evidence; no automatic retry is added.

## Root-Cause Hypotheses

1. Confirmed shutdown defect: shutdown joined workers before the face thread,
   which could create a worker after its join check. Face-first quiescence
   removed the original post-`GS_GUI_EXIT` terminate signature.
2. Disproved as a complete fix: making the auto-MAVLink worker owned and moving
   its commands onto the GTK thread did not remove the 5% pre-shutdown abort.
3. Current controlling hypothesis: two `ServiceUser` response workers execute
   lightweight telemetry and command callbacks concurrently against shared
   Ground Station runtime/UI state. Serialize this control plane on the Face
   thread; keep compute-heavy object-detection provider workers unchanged.
4. Sanitizer diagnosis was attempted, but parallel debug compilation exceeded
   local memory. Do not repeat it unless using a separate low-parallel build
   environment with enough time and disk.

## Design

1. Set `m_done`, join authority refresh and face threads, stop Core/io work,
   then join/stop workers that those producers can create.
2. Own and join the auto-MAVLink worker, route its GUI actions through the GTK
   main context, and serialize Ground Station user callbacks on the Face thread.
3. Add payload-free Targeted phase logs around queue, dispatch, response, and
   timeout; add UAV command phase logs around local admission and callbacks.
4. Extend the isolation parser with both observed abort signatures and a
   command-stage summary.
5. Build and run focused tests, then execute control-only 0% and 5% campaigns,
   five runs each, no retries.

## Validation

- Static thread-order and sensitive-field scan.
- Affected UAV build plus full C++ unit tests.
- Focused and full Python tests.
- Ten control-only MiniNDN runs with zero abort marker.
- Strict Spec Kit audit/converge, GSD health, CodeGraph sync, clean git.

## Rollback

Revert the shutdown-order and diagnostic commit. No protocol, persistence, or
data migration is involved.
