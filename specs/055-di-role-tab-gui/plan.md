# Plan: NDNSF-DI Three-Role Tk Console

## Context

The existing GUI already provides policy generation, policy editing, model split
inspection, certificate helpers, and a deployment runner. The deployment runner
is still command-centric and not yet the operator experience the project needs.

The Python wrappers already expose the runtime roles:

- `ndnsf.service.ServiceController`
- `ndnsf.service.ServiceProvider`
- `ndnsf.service.ServiceUser`

The preferred design is to build a GUI role controller layer around these
wrappers. The GUI should use direct Python APIs where practical, but retain a
subprocess fallback for long-running app-specific role scripts.

## Architecture

```text
DistributedInferenceGui
  ttk.Notebook
    UserRoleTab
    ProviderRoleTab
    ControllerRoleTab
    Policy Editor
    Project Wizard
    Model Split
    Certificates

RoleRuntimeController
  state machine
  config validation
  env overlay
  background thread or subprocess
  log queue
  stop/restart

RoleConfig
  shared fields
  controller config
  provider config
  user config
  env knobs
  request defaults

UserRequestPanel
  normal service request
  targeted request
  collaboration request
  payload codecs
  response rendering
```

## Direct API Versus Subprocess

Default path:

- Controller tab instantiates `ServiceController` and calls `start_background()`
  or runs it in a managed thread.
- Provider tab instantiates `ServiceProvider`, registers the chosen handler, and
  starts it in a managed thread.
- User tab instantiates `ServiceUser`, calls `start()`, and dispatches service
  requests through `request_service()` or `request_collaboration_async()`.

Fallback path:

- If a selected provider/user mode is an existing script that owns complex app
  setup, the tab may run that script as a subprocess using the existing command
  builder pattern.
- The fallback must still show status/logs and must not be the only way to use
  the request panel.

Rationale:

- Direct API gives a real request/response GUI.
- Subprocess fallback avoids forcing every old app entrypoint into one GUI
  thread model immediately.

## Role Lifecycle

Each tab uses the same lifecycle:

```text
stopped -> starting -> running -> stopping -> stopped
                         |
                         -> failed
                         -> exited
```

Rules:

- `Run` is disabled while starting/running.
- `Stop` is enabled only while starting/running.
- exceptions are logged and move the role to failed.
- GUI updates happen through `after()` or a thread-safe queue.
- application exit calls stop on all role controllers.

## Config Model

Create structured dataclasses:

```text
SharedNdnsfConfig
ControllerTabConfig
ProviderTabConfig
UserTabConfig
NdnsfSvsEnvConfig
NdnsfDiRuntimeConfig
UserRequestConfig
ThreeRoleGuiProfile
```

The existing `RuntimeGuiProfile` can either be migrated or kept as a legacy
loader. New profiles should be versioned:

```json
{
  "version": 2,
  "controller": {},
  "provider": {},
  "user": {},
  "env": {}
}
```

## GUI Layout

### Controller tab

Sections:

- identity and security
- policy and permissions
- bootstrap token table
- environment
- lifecycle/logs

Primary buttons:

- Run Controller
- Stop Controller
- Validate Policy
- Open Token File
- Add/Update Token
- Save Profile

### Provider tab

Sections:

- identity and security
- service advertisement
- provider role/runtime
- DI runtime and fragment inventory
- ACK metadata
- NDNSD/probing/SVS
- lifecycle/logs

Primary buttons:

- Run Provider
- Stop Provider
- Publish Service Info
- Refresh Runtime State
- Validate Config

### User tab

Sections:

- identity and security
- request settings
- service discovery/permissions
- request payload
- collaboration/deployment options
- response/logs

Primary buttons:

- Run User
- Stop User
- Refresh Permissions
- Discover Services
- Send Request
- Send Collaboration Request
- Clear Response

## Validation Strategy

Non-display tests:

- profile serialization and migration from old `RuntimeGuiProfile`
- required-field validation
- env overlay redaction
- role lifecycle with fake runtime objects
- independent start/stop of all three fake roles
- user request dispatch with fake `ServiceUser`
- JSON field parsing for ACK metadata, DI runtime, collaboration roles,
  dependencies, and env knobs

Manual/MiniNDN validation after implementation:

1. start local NFD or MiniNDN environment
2. run Controller tab
3. run Provider tab with `/HELLO` echo handler or NativeTracer handler
4. run User tab
5. send `/HELLO` request and verify response
6. repeat with DI collaboration request when full runtime artifacts are present

## Risks

- **In-process runtime lifecycle**: C++/pybind Face shutdown may not restart
  cleanly in the same process. Mitigation: preserve subprocess fallback and
  make restart tests explicit.
- **Too many fields**: exposing every knob can overwhelm users. Mitigation:
  use Basic/Advanced sections; keep important fields visible and env knobs in
  collapsible advanced area.
- **Secret leakage**: tokens are useful in GUI but dangerous in logs/profiles.
  Mitigation: redact visible logs and make profile token persistence explicit.
- **App-specific handlers**: a generic provider tab cannot know every DI app
  handler. Mitigation: support handler modes and custom script fallback.

