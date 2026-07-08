# Implementation Plan: Deployment ACK Core Readiness

## Summary

Replace the inline deployment ACK observer parsing in `ServiceUser.deploy_service`
with a small reusable helper that uses `parse_ack_metadata`,
`ProviderCapabilityHint`, and `ServiceDiscoveryRecord` for ready providers, while
preserving explicit provisioning ACK support for deployment setup.

## Design

1. Add `_deployment_roles_from_ack_candidate(candidate)`.
2. Parse ACK payload with core `parse_ack_metadata`.
3. For positive ACKs:
   - if `providerCapabilityHint` exists, require
     `ServiceDiscoveryRecord.ready_for_new_request()`;
   - otherwise keep legacy positive ACK behavior.
4. For negative ACKs:
   - accept only `MODEL_UNAVAILABLE` / `ModelUnavailable` as provisioning;
   - use `provisioningRole` first, then `roles`.
5. Update `deploy_service` observer to call the helper.

## Verification

```bash
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_deployment_ack_core_readiness.py
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_service_discovery.py
```

