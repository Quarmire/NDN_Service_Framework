# Plan: Tk GUI For NDNSF-DI Operators

## Design

NDNSF-DI already has a Tk GUI for policy generation, policy editing, model split
inspection, certificate helpers, and simple role launch buttons. This phase
turns the role launch area into a practical operator control panel.

The implementation is intentionally process-based:

1. The GUI builds commands for existing controller, provider, and user scripts.
2. A small process supervisor starts/stops those commands with stdout/stderr
   merged into the GUI log.
3. Each role tab owns editable configuration fields and a text status.
4. A reusable JSON profile stores role settings and can be loaded on another
   machine or later run.

This keeps the GUI thin and avoids changing APP-level Python APIs or the C++
runtime. The provider and user remain configurable through GUI fields such as
policy path, provider ID, role list, service name, timeout, generated policy
directory, and extra args.

## Data Model

- `RoleProcessState`: label, command, process handle, status, return code.
- `RuntimeRoleProfile`: role name, example app, policy config,
  generated-policy-dir, group override, provider ID, roles, service, timeout
  fields, extra args.
- `RuntimeGuiProfile`: controller, provider, and user role profiles.

## GUI Changes

- Add per-role status labels.
- Add Start, Stop, Restart, and Show Command buttons to each role tab.
- Add Load Profile, Save Profile, Start All, Stop All, and Clear Logs buttons to
  the deployment runner.
- Use `shlex.split` for extra args instead of naive whitespace splitting.
- Keep process output in a shared deployment log with role labels.

## Validation

- Add Python tests for profile roundtrip and command construction.
- Add process-supervisor tests using short Python one-liners, not NDNSF network
  processes.
- Run py_compile on the GUI and test file.
- Run focused Python tests.
- Run `git diff --check`.
- Sync/check CodeGraph after edits.
