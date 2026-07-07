# Tasks: NDNSF-DI Three-Role Tk Console

## Phase 0 - Guardrails

- [ ] T001 Confirm current dirty worktree ownership before editing
      `NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py` and
      `tests/python/test_ndnsf_di_tk_gui.py`.
- [ ] T002 Run `codegraph status .` and, if needed, `codegraph sync .`.
- [ ] T003 Preserve existing policy editor, project wizard, model split,
      certificate helper, and old profile loading behavior.

## Phase 1 - Config Data Model

- [ ] T004 Add dataclasses for `SharedNdnsfConfig`, `NdnsfSvsEnvConfig`,
      `ControllerTabConfig`, `ProviderTabConfig`, `UserTabConfig`,
      `UserRequestConfig`, and `ThreeRoleGuiProfile`.
- [ ] T005 Add profile versioning and migration from existing
      `RuntimeGuiProfile`.
- [ ] T006 Add JSON parsing helpers for advanced fields:
      ACK metadata, provider runtime metadata, fragment inventory,
      collaboration roles, key scopes, dependencies, artifact Data names,
      scope-key Data names, role scopes, and NDNSD metadata.
- [ ] T007 Add validation helpers for required names, paths, integer fields,
      JSON fields, and service names.
- [ ] T008 Add secret redaction helpers for tokens and token-file contents.

## Phase 2 - Role Runtime Controller

- [ ] T009 Add a reusable `RoleRuntimeController` state machine with
      `run()`, `stop()`, `restart()`, `status`, `last_error`, and log queue.
- [ ] T010 Support direct Python runtime mode using wrapper factories for
      `ServiceController`, `ServiceProvider`, and `ServiceUser`.
- [ ] T011 Preserve subprocess mode using the existing command-building pattern
      for app-specific scripts.
- [ ] T012 Add environment overlay support that applies selected NDNSF/SVS env
      knobs for a role and restores them after stop when possible.
- [ ] T013 Ensure all Tk updates happen through `after()` or a queue, never
      directly from worker threads.
- [ ] T014 Add clean application-exit shutdown for all running role controllers.

## Phase 3 - Controller Tab

- [ ] T015 Build `ControllerRoleTab` with sections for identity/security,
      policy/permissions, token file, env knobs, and logs.
- [ ] T016 Wire fields to `ServiceController(controller_prefix, policy_file,
      trust_schema, bootstrap_identities, serve_certificates,
      bootstrap_token_file)`.
- [ ] T017 Add token-file helper UI: open file, create if missing, add/update
      name-token entry, save file, show redacted table.
- [ ] T018 Add Validate Policy and Show Effective Config buttons.
- [ ] T019 Add Run/Stop/Restart buttons and status label.

## Phase 4 - Provider Tab

- [ ] T020 Build `ProviderRoleTab` with sections for identity/security,
      service/roles, ACK metadata, DI runtime, NDNSD/probing, env knobs, and
      logs.
- [ ] T021 Wire fields to `ServiceProvider(provider_id, group, controller,
      provider_prefix, trust_schema, handler_threads, ack_threads,
      serve_certificates, bootstrap_token)`.
- [ ] T022 Add provider handler modes: echo, static response, NativeTracer
      script fallback, custom command fallback, and dry-run.
- [ ] T023 Add ACK handler configuration from status/message/metadata JSON.
- [ ] T024 Add NDNSD service-info publishing controls:
      service lifetime and metadata JSON.
- [ ] T025 Add provider probing controls if the wrapper/runtime exposes
      `startProviderProbing` or equivalent; otherwise show disabled with a clear
      message.
- [ ] T026 Add DI runtime fields for runtime profile, service manifest, native
      plan, fragment inventory, artifact cache, memory/compute profile, and
      deployment id.

## Phase 5 - User Tab

- [ ] T027 Build `UserRoleTab` with sections for identity/security,
      permissions/discovery, request settings, payload, collaboration options,
      response, and logs.
- [ ] T028 Wire fields to `ServiceUser(group, controller, user, trust_schema,
      permission_wait_ms, handler_threads, ack_threads, adaptive_admission,
      serve_certificates, bootstrap_token)`.
- [ ] T029 Add Refresh Permissions using `get_allowed_services()`.
- [ ] T030 Add Discover Services using `get_ndnsd_services()` when NDNSD is
      enabled.
- [ ] T031 Add payload codecs: text, JSON, hex, and file.
- [ ] T032 Add normal request action using `request_service()` in a background
      task.
- [ ] T033 Add async request action using `request_service_async()` when the
      user runtime is started.
- [ ] T034 Add collaboration request action using `request_collaboration()` or
      `request_collaboration_async()` from JSON roles/dependencies fields.
- [ ] T035 Render response status, message, payload preview, elapsed time, and
      errors in the response panel.

## Phase 6 - Main Window Integration

- [ ] T036 Replace or reorganize the old deployment runner so the top-level
      operational tabs are exactly `User`, `Provider`, and `Controller`.
- [ ] T037 Keep policy/model/certificate tools available as secondary tabs or
      menu entries.
- [ ] T038 Add Load Profile and Save Profile for the full three-role profile.
- [ ] T039 Add optional Run All and Stop All buttons that delegate to each tab's
      own lifecycle path.
- [ ] T040 Add status bar summary for all three roles.

## Phase 7 - Tests

- [ ] T041 Extend `tests/python/test_ndnsf_di_tk_gui.py` for new profile
      serialization and migration.
- [ ] T042 Add fake runtime factories and test each role starts only after Run.
- [ ] T043 Test simultaneous fake Controller/Provider/User lifecycle.
- [ ] T044 Test user request dispatch and response rendering with fake
      `ServiceUser`.
- [ ] T045 Test JSON-field validation and error rendering.
- [ ] T046 Test secret redaction in logs/profile preview.
- [ ] T047 Test command/subprocess fallback construction for provider/user app
      scripts.

## Phase 8 - Validation

- [ ] T048 Run:
      `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper python3 -m py_compile NDNSF-DistributedInference/ndnsf_distributed_inference/gui.py tests/python/test_ndnsf_di_tk_gui.py`.
- [ ] T049 Run:
      `PYTHONPATH=NDNSF-DistributedInference:pythonWrapper python3 tests/python/test_ndnsf_di_tk_gui.py`.
- [ ] T050 Run `git diff --check`.
- [ ] T051 Run `codegraph sync . && codegraph status .`.
- [ ] T052 Optional integration smoke: start local controller/provider/user
      with `/HELLO` and send one GUI request.

## Phase 9 - Documentation

- [ ] T053 Update `docs/NDNSF-DI-runtime-workflow.md` with the GUI as an
      operator entrypoint, while keeping CLI commands as reproducible evidence
      paths.
- [ ] T054 Add a short GUI profile example under
      `examples/python/NDNSF-DistributedInference/`.
- [ ] T055 Document direct API mode versus subprocess fallback and when to use
      each.

