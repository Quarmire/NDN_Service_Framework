# Spec: NDNSF-DI Three-Role Tk Console

**Feature**: `055-di-role-tab-gui`  
**Status**: Draft  
**Created**: 2026-07-07  

## Summary

Replace the current role-runner area of the NDNSF-DI Tk GUI with a practical
three-tab operator console:

```text
User | Provider | Controller
```

Each tab owns its own configuration, status, log, run button, stop button, and
runtime object. A role starts only when the operator clicks **Run** on that tab.
The three roles may run at the same time in one GUI session. The User tab also
provides a request panel for sending a service request and displaying the
response.

The GUI is an operator layer only. It must not introduce new NDNSF protocol
messages, bypass NAC-ABE, bypass controller permissions, or replace the
existing Python wrapper and runtime APIs.

## User Scenarios

### P1: Start the controller from the Controller tab

The operator configures controller prefix, policy file, trust schema, bootstrap
token file, bootstrap identities, certificate serving, and runtime environment.
The controller is not started until the operator clicks **Run Controller**. The
tab displays ready/running/failure state and controller logs.

### P1: Start a provider from the Provider tab

The operator configures provider identity, provider prefix/id, controller name,
sync group, trust schema, bootstrap token, provider permissions/policy context,
service name, service roles, ACK behavior, runtime metadata, NDNSD publishing,
provider probing, and DI-specific runtime profile. The provider starts only
when **Run Provider** is clicked and may run while controller and user tabs are
also active.

### P1: Start a user and request a service from the User tab

The operator configures user identity, controller name, sync group, trust
schema, bootstrap token, permission wait, request strategy, ACK timeout, full
request timeout, service name, request payload, and optional collaboration
roles/dependencies. The user starts only when **Run User** is clicked. After
the user is running, the operator can send a request and see ACK summary,
selected provider, response status, response payload, latency, and errors.

### P2: Save and reload a complete GUI profile

The operator can save all three tabs' configuration to one JSON profile and
load it later. Loading a profile must not auto-start any role.

### P2: Run all roles for a local demo without hiding individual control

The GUI may provide optional **Run All** and **Stop All** buttons, but these
must call the same per-tab lifecycle paths as the individual Run/Stop buttons.
Each tab remains independently visible and controllable.

## Requirements

- **REQ-055-001**: The GUI MUST contain top-level `User`, `Provider`, and
  `Controller` tabs.
- **REQ-055-002**: No role may start during GUI launch or profile load.
- **REQ-055-003**: Each role MUST start only when its own Run button is clicked.
- **REQ-055-004**: The three roles MUST be able to run simultaneously in one
  GUI session when the underlying runtime permits it.
- **REQ-055-005**: Each role tab MUST show textual status: `stopped`,
  `starting`, `running`, `stopping`, `exited`, or `failed`.
- **REQ-055-006**: Each role tab MUST show role-scoped logs and errors.
- **REQ-055-007**: Each role tab MUST expose common NDN runtime fields:
  controller prefix, sync group, trust schema, certificate serving, bootstrap
  token or token file, and relevant identity/prefix fields.
- **REQ-055-008**: The Controller tab MUST expose policy file, bootstrap token
  file, bootstrap identities, controller prefix, trust schema, and certificate
  serving.
- **REQ-055-009**: The Controller tab SHOULD provide token-file helpers: open,
  create if missing, add/update name-token entries, and view redacted token
  table.
- **REQ-055-010**: The Provider tab MUST expose provider id, provider prefix,
  controller, sync group, trust schema, bootstrap token, service name, roles,
  handler/ACK thread counts, ACK status/metadata, and certificate serving.
- **REQ-055-011**: The Provider tab SHOULD expose DI runtime fields: provider
  memory/compute profile, fragment inventory file, residency hints,
  deployment id, runtime profile path, service manifest path, native execution
  plan path, artifact cache directory, and provider probing toggle/interval.
- **REQ-055-012**: The User tab MUST expose user identity, controller, sync
  group, trust schema, bootstrap token, permission wait, service name, request
  strategy, ACK timeout, request timeout, adaptive admission toggle, request
  payload, and payload encoding.
- **REQ-055-013**: The User tab MUST provide a request/response panel that can
  send a normal service request and display success, status message, payload,
  selected provider evidence when available, and elapsed time.
- **REQ-055-014**: The User tab SHOULD support collaboration request inputs:
  roles, key scopes, dependencies, artifact Data names, scope-key Data names,
  role scopes, and deployment id.
- **REQ-055-015**: The GUI MUST expose important NDNSF/NDNSF-DI environment
  knobs without requiring command-line editing: `NDNSF_ENABLE_NDNSD`,
  `NDNSF_DISABLE_NDNSD`, `NDNSF_SVS_EXPECTED_RPS`,
  `NDNSF_SVS_PUBLICATION_FETCH_RETRIES`,
  `NDNSF_SVS_PUBLICATION_FETCH_INNER_RETRIES`,
  `NDNSF_SVS_PUBLICATION_FETCH_LIFETIME_MS`,
  `NDNSF_SVS_PUBLICATION_FETCH_BACKOFF_MS`,
  `NDNSF_SVS_PUBLICATION_FETCH_MAX_BACKOFF_MS`,
  `NDNSF_SVS_PUBLICATION_FETCH_WINDOW`,
  `NDNSF_SVS_MAX_SUPPRESSION_MS`, `NDNSF_SVS_PERIODIC_SYNC_MS`,
  `NDNSF_SVS_PARALLEL_SYNC`, `NDNSF_SVS_PARALLEL_WORKERS`,
  `NDNSF_SVS_PARALLEL_QUEUE`, `NDNSF_SVS_PARALLEL_PRODUCTION`,
  `NDNSF_SVS_SYNC_BATCHING`, and `NDNSF_SVS_SYNC_BATCH_MS`.
- **REQ-055-016**: The GUI MUST save/load a profile that includes all role
  fields, environment knobs, and request-panel defaults.
- **REQ-055-017**: The GUI MUST validate obvious invalid inputs before Run:
  empty identity, empty group, missing policy file, missing trust schema,
  invalid integer fields, invalid JSON fields, and empty service name.
- **REQ-055-018**: The GUI MUST not print secrets in logs. Bootstrap tokens and
  token files must be redacted in visible logs and saved only when the user
  explicitly chooses to save them in the profile.
- **REQ-055-019**: The implementation MUST preserve the existing policy editor,
  policy wizard, model split, and certificate helper features unless the user
  explicitly asks to remove them.
- **REQ-055-020**: The implementation MUST have non-display tests for profile
  serialization, config validation, command/direct-runtime construction,
  lifecycle state transitions, and user request task dispatch.

## Important Configuration Surface

### Shared fields

- controller prefix/name
- sync group
- trust schema path
- certificate serving enabled/disabled
- bootstrap token
- identity/prefix
- handler/ACK worker counts where applicable
- NDNSD enable/disable
- SVS publication fetch retry/window/lifetime/backoff knobs
- SVS suppression, periodic sync, parallel sync/production, batching knobs

### Controller fields

- controller prefix
- policy file
- bootstrap token file
- bootstrap identities list
- trust schema
- serve certificates
- token table helper fields: identity name, token, enabled/reusable flag,
  notes/comment

### Provider fields

- provider id
- provider prefix
- provider identity preview
- controller
- sync group
- trust schema
- bootstrap token
- service name
- roles
- ACK enabled/status/message/metadata JSON
- handler mode: echo, NativeTracer, llama/server, custom script, or dry-run
- DI runtime profile
- native execution plan
- service manifest
- artifact cache directory
- fragment inventory JSON
- runtime telemetry JSON
- memory/capacity profile JSON
- NDNSD service metadata JSON
- provider peer probing toggle and interval

### User fields

- user identity
- controller
- sync group
- trust schema
- bootstrap token
- permission wait
- service name
- request strategy: first responding, random, all selected, custom/select
- ACK timeout
- full request timeout
- adaptive admission toggle
- request mode: normal service, targeted service, collaboration, deployment id
- payload encoding: text, hex, JSON, file
- request payload
- collaboration roles JSON
- key scopes JSON
- dependencies JSON
- artifact Data names JSON
- scope-key Data names JSON
- role scopes JSON
- deployment id

## Non-Goals

- No new wire protocol.
- No replacement of ServiceController, ServiceProvider, or ServiceUser.
- No security bypass for demo convenience.
- No GUI-only behavior that cannot be reproduced through the Python runtime
  wrapper or experiment scripts.
- No display-dependent tests in CI.

## Success Criteria

- **SC-055-001**: A test can build a profile with all three tabs, serialize it,
  load it, and recover all key fields.
- **SC-055-002**: A test can instantiate tab controllers with fake
  ServiceController/ServiceProvider/ServiceUser factories and verify each role
  starts only after its Run action.
- **SC-055-003**: A test can start fake controller, provider, and user roles
  simultaneously and stop them independently.
- **SC-055-004**: A test can dispatch a fake user request and verify response,
  error, and elapsed-time rendering.
- **SC-055-005**: Existing GUI helper tests continue to pass.
- **SC-055-006**: `py_compile` passes for the GUI module and focused tests.

