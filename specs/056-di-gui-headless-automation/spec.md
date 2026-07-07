# Feature Spec: NDNSF-DI GUI Headless Automation

## Goal

The NDNSF-DI Tk GUI should be testable without manual clicking. The same
configuration used by the `User`, `Provider`, and `Controller` tabs must also
drive a headless command-line mode for scripted validation, CI-style smoke
tests, and MiniNDN launchers.

## User-Facing Behavior

- A normal GUI launch still opens the Tk window.
- `--headless` or `-headless` runs without creating a Tk root window.
- `--controller-auto-run`, `--provider-auto-run`, and `--user-auto-run`
  start only the selected roles.
- Single-dash underscore aliases such as `-user_auto_run` and
  `-user_config=user1.config` are accepted for shell-friendly testing.
- `--profile` loads the full three-role GUI profile.
- `--user-config`, `--provider-config`, and `--controller-config` merge
  role-specific JSON files into the profile.
- `--runtime-mode fake` runs an in-process fake runtime for deterministic
  non-network tests.
- `--runtime-mode direct` uses the real Python NDNSF wrapper classes and is the
  path to use inside an already prepared NFD/MiniNDN environment.
- `--send-user-request` sends the configured User request after the User role is
  running.
- `--output-json` writes machine-readable status, response, errors, and logs.

## Non-Goals

- Headless mode does not replace MiniNDN campaign scripts.
- Headless mode does not add a new NDNSF protocol path.
- Headless mode does not bypass authentication or controller permission logic
  in direct runtime mode.

## Acceptance Criteria

- Existing GUI helper tests still pass.
- Headless fake mode can start Controller, Provider, and User and issue a
  request without a display server.
- Role-specific config files can override one tab without touching the others.
- The MiniNDN GUI launcher can run a headless preflight smoke before any
  interactive GUI launch.
