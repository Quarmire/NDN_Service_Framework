# Feature Specification: Deployment ACK Core Readiness

**Feature Branch**: `060-deployment-ack-core-readiness`

**Created**: 2026-07-08

**Status**: Draft

## User Story

As an NDNSF deployment user, I need deployment ACK role capture to use core
readiness metadata for ready providers while still accepting explicit
provisioning ACKs for deployment setup.

## Requirements

- **FR-001**: Deployment ACK parsing MUST use core ACK metadata parsing instead
  of an ad-hoc semicolon parser.
- **FR-002**: A positive ACK with typed `ProviderCapabilityHint` MUST only record
  roles when the core discovery record is ready for a new request.
- **FR-003**: A positive legacy ACK without typed hints MUST keep existing role
  capture behavior.
- **FR-004**: A negative ACK MUST only be recorded as provisioning when it
  explicitly reports `MODEL_UNAVAILABLE` or legacy `ModelUnavailable`.
- **FR-005**: Draining/unready typed hints MUST not be treated as ready
  deployment assignments.

## Non-Goals

- Do not change the deployment wire protocol.
- Do not remove explicit provisioning support.
- Do not move DI deployment policy into core.

## Success Criteria

- Focused unit tests cover typed ready, typed draining, legacy ready, explicit
  provisioning ACK, and non-provisioning negative ACK.

