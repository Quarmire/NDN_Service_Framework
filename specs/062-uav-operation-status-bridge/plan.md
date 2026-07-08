# Implementation Plan: UAV Operation Status Bridge

## Summary

Add thin C++ bridge helpers in `NDNSF-UAV-APP/shared/UavProtocol.*` that convert
existing UAV domain states into core runtime envelopes. The core remains
service-neutral; UAV keeps all domain semantics and only exports reusable facts
through `ServiceOperationStatus` and `DataProductReference`.

## Design

1. Add `RecordingDataProductState::toDataProductReference()` for completed
   recording objects.
2. Add overloaded `toServiceOperationStatus()` helpers for:
   - `FlightCommandState`
   - `RecordingDataProductState`
   - `MissionState`
   - `MissionProgressState`
3. Map UAV states into the core lifecycle vocabulary:
   - queued/idling work -> `QUEUED`
   - active command/mission/recording work -> `RUNNING`
   - uploaded mission waiting for start -> `WAITING_INPUT`
   - successful terminal state -> `DONE`
   - failed terminal state -> `FAILED`
   - cancelled mission -> `CANCELED`
   - command timeout -> `EXPIRED`
4. Preserve UAV-specific context in `reasonCode` and `message` because the C++
   core operation status does not currently expose a metadata map.
5. Keep tests focused on round-tripping through the existing core
   `ServiceProvider::makeServiceOperationStatusPayload()` parser.

## Verification

```bash
./waf build --target=unit-tests
./build/unit-tests --run_test=UavProtocolState
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper python3 tests/python/test_ndnsf_app_core_envelope_migration.py
```
