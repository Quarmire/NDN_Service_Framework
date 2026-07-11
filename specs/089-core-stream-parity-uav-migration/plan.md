# Implementation Plan: Core Stream Parity And UAV Migration

## Constitution Check

No invocation/security naming changes. Core owns only app-neutral stream state;
UAV owns codec and flight semantics. C++/Python unit parity and MiniNDN are
required.

## Design

1. Freeze JSON parity vectors for session, reorder, duplicate, gap, skip,
   overflow, metrics, malformed and adaptive inputs.
2. Complete C++ pending count/bytes and overflow metrics where parity exposes a
   gap.
3. Bind the C++ structs/state classes through `_ndnsf`.
4. Convert `pythonWrapper/ndnsf/streaming.py` buffers/fetcher into thin native
   adapters while retaining Python value objects and health presentation.
5. Route UAV generic state through Core; retain all domain policy.
6. Prove static objects use SegmentFetcher-style exact-name retrieval.

## Threading

Each buffer instance is caller-serialized or internally locked as documented;
Python wrappers release no object while a native call is active. No callback is
invoked while the native mutex is held.
