# Spec: Tk GUI For NDNSF-DI Operators

## Goal

Provide a Python Tk interface for running NDNSF-DI controller, provider, and
user roles without forcing operators to remember every command-line option.
The GUI must keep the existing APP/runtime APIs as the source of truth and
should launch existing role scripts rather than inventing a new wire protocol.

## Users

- Controller operator: starts the service controller, signs participant
  certificates, and watches controller logs.
- Provider operator: configures provider ID, roles, generated policy directory,
  service policy, and starts/stops a provider role process.
- User operator: configures user request parameters and starts/stops user
  requests from the GUI.
- Developer: runs MiniNDN/regression commands and validates policy or example
  commands from the same GUI.

## Requirements

- The GUI SHALL expose separate Controller, Provider, and User tabs.
- Each role tab SHALL have editable configuration fields for the command inputs
  needed by that role.
- Each role tab SHALL provide buttons for preparing, starting, stopping, and
  restarting that role.
- The GUI SHALL show role status as text: stopped, starting, running, exited,
  or failed, including process return code when known.
- The GUI SHALL route stdout/stderr into a visible log pane with role labels.
- The GUI SHALL support Start All and Stop All buttons for the configured
  controller, provider, and user roles.
- The GUI SHALL support saving and loading the role configuration as JSON so
  user/provider/controller setups can be reused.
- The GUI SHALL keep charts, model analysis, policy editing, and certificate
  helper features intact.
- The GUI SHALL use standard Python/Tk dependencies only.
- The implementation SHALL be covered by focused non-display tests for profile
  serialization, command building, and process-status state transitions.

## Non-Goals

- No new NDNSF-DI protocol messages.
- No change to permission, NAC-ABE, or token logic.
- No app-specific deep integration with one model runner. The GUI launches the
  current role scripts and lets each script own its app-level handler.
- No attempt to make Tk tests require an X display.

## Acceptance Criteria

- `python3 -m ndnsf_distributed_inference.gui` still launches the GUI when Tk is
  available.
- A user can save a profile, load it back, and see role fields restored.
- Controller/provider/user roles can be started, stopped, and restarted from
  buttons.
- Start All starts the configured controller/provider/user commands; Stop All
  terminates still-running role processes.
- Tests pass without opening a GUI window.
