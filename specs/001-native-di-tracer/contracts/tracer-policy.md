# Contract: Tracer Policy Bundle

## Purpose

The tracer policy bundle is the planning-plane contract between Python NDNSF-DI tools and native C++ execution.

## Required Outputs

- `trust-schema.conf`
- `controller.policies`
- `service-manifest.json`
- `service-manifest.json.sha256`
- `native-execution-plan.json`
- `native-execution-plan.json.sha256`

## Acceptance Rules

- Service name is a unified NDNSF service name.
- Role/provider coverage is complete.
- Source inputs and dependency edges are explicit.
- The final role declares `final-response`.
- Generated files are reproducible enough for unit/smoke tests to consume without manual edits.
