# Tasks: UAV Operation Status Bridge

- [x] T001 Review existing UAV state structures and core operation-status C++
  helper.
- [x] T002 Add durable Spec062 artifacts.
- [x] T003 Add `RecordingDataProductState::toDataProductReference()`.
- [x] T004 Add UAV `toServiceOperationStatus()` helpers for flight command,
  recording, mission part, and mission progress.
- [x] T005 Add C++ unit tests that round-trip mapped statuses through the core
  serializer/parser.
- [x] T006 Update the core/app boundary document.
- [x] T007 Build `unit-tests`.
- [x] T008 Run `UavProtocolState`.
- [x] T009 Run Python app-core envelope migration regression.
