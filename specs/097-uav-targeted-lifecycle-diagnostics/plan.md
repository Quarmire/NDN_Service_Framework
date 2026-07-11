# Implementation Plan

## Constitution Check

- Canonical V2 Targeted invocation and unified service names remain unchanged.
- NAC-ABE, permission, token, replay, and provider checks stay in the path.
- CodeGraph established callers and thread ownership before edits.
- MiniNDN is the final lifecycle/network verification path.
- Failed commands and runs remain evidence; no automatic retry is added.

## Root-Cause Hypotheses

1. Most likely: shutdown joins workers before joining the face thread, which can
   create a worker after its join check; prediction: face-first quiescence
   removes the post-GUI SIGABRT.
2. Detached window automation accesses destroyed state; prediction: aborts
   persist after face-first shutdown or sanitizers reveal use-after-free.
3. Decoder/YOLO shutdown races independently; prediction: aborts correlate only
   with video/YOLO-active runs.

## Design

1. Set `m_done`, stop Core/io work, join authority refresh and face threads,
   then join/stop workers that those producers can create.
2. Add payload-free Targeted phase logs around queue, dispatch, response, and
   timeout; add UAV command phase logs around local admission and callbacks.
3. Extend the isolation parser with `lifecycleAbort` and command-stage summary.
4. Build and run focused tests, then execute control-only 0% and 5% campaigns,
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
