# Tasks: Runtime Doctor and Usability Harness

- [x] T001 Add a JSON runtime profile for the HELLO/token-bootstrap path.
- [x] T002 Add a stdlib-only `tools/ndnsf_runtime.py doctor` command.
- [x] T003 Add JSONL structured events and resolved config output.
- [x] T004 Add missing-token-file generation from policy identities with
  8-character tokens.
- [x] T005 Add Python unit tests for doctor token generation and missing-token
  diagnostics.
- [x] T006 Add shared shell regression helpers for NFD startup, log waiting,
  cleanup, and log tailing.
- [x] T007 Migrate token certificate bootstrap regression to use the shared
  helper for NFD startup and log waits.
- [x] T008 Document the runtime doctor workflow in README.
- [x] T009 Validate with unit tests, build, token bootstrap regression, and
  aggregate security regression.
- [x] T010 Add a DI NativeTracer runtime profile for the Qwen tiny proportional
  MiniNDN harness.
- [x] T011 Extend the doctor with NativeTracer artifact, topology, binary, and
  command preflight checks.
- [x] T012 Add regression coverage for DI profile resolution and structured
  preflight events.
- [x] T013 Document the DI runtime doctor workflow in README and quickstart.
- [x] T014 Let the NativeTracer MiniNDN harness read runtime profile or resolved
  doctor JSON defaults directly.
- [x] T015 Add regression coverage proving the profile drives local
  NativeTracer execution defaults.
