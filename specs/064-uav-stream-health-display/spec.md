# Feature Specification: UAV Stream Health Display

**Feature Branch**: `064-uav-stream-health-display`

**Created**: 2026-07-08

**Status**: Draft

## User Story

As a UAV ground-station operator and NDNSF developer, I need the UAV GUI to
display the reusable core stream-health summary alongside existing adaptive
video details, so live-video health can be interpreted through the same core
stream vocabulary without removing UAV-specific bitrate/FEC/ROI context.

## Requirements

- **FR-001**: `VideoAdaptiveState` MUST expose a concise display summary derived
  from core `StreamHealth`.
- **FR-002**: Ground-station GUI text SHOULD show core stream health next to the
  existing adaptive-video compact summary.
- **FR-003**: The migration MUST preserve existing adaptive-video text,
  bitrate policy, decoder behavior, FEC behavior, and GUI layout structure.
- **FR-004**: Unit tests MUST verify that the displayed health summary reflects
  core `StreamHealth` state, reason, window, and gap metrics.

## Non-Goals

- Do not redesign the GTK layout.
- Do not change H264/FEC/ROI or bitrate control policy.
- Do not change stream fetching behavior.
