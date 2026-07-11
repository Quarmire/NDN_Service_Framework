# Private Operation Authorization Evidence

Date: 2026-07-11

## Boundary

- Public object calls use `/NDNSF/DistributedRepo/Object/v1/*`.
- Repo peer calls use `/NDNSF/DistributedRepo/Internal/v1/*`.
- Service name and payload operation must map to the same versioned endpoint.
- Internal handlers reject missing requester identities and identities outside
  the configured Repo provider namespace/explicit peer allowlist.
- Client write-transaction substeps remain under public `INSERT`; explicit
  `PEER_*` operations are required to enter peer-only reserve/release/commit.
- The generic Repo policy grants public services to the object user and Repo
  providers, but grants Internal services only to Repo provider identities.

## Verification

```text
PYTHONPATH=pythonWrapper:NDNSF-DistributedRepo/pythonWrapper:NDNSF-DistributedInference \
  python3 tests/python/test_ndnsf_repo_core_discovery_selection.py
11 tests passed

PYTHONPATH=pythonWrapper:NDNSF-DistributedRepo/pythonWrapper:NDNSF-DistributedInference \
  python3 -m unittest discover -s tests/python -p 'test_ndnsf_repo_*.py'
86 tests passed
```

The focused negatives cover operation/service mismatch, missing peer identity,
ordinary user identity on an Internal service, and authenticated Repo provider
identity acceptance. Both shipped YAML deployments parse with the expanded
versioned service list.
