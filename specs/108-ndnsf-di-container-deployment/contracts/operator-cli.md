# Operator CLI Contract: `ndnsf-di-deploy`

## Purpose

Provide one non-interactive, scriptable surface over OCI release handling and the `docker-compose` and `slurm-apptainer` adapters. It does not replace Docker, Slurm, Apptainer, NFD, or NDNSF-DI; it validates policy, invokes them, and preserves evidence.

## Global behavior

```text
ndnsf-di-deploy [--json] [--verbose] COMMAND [OPTIONS]
```

- Exit `0`: requested operation completed and its contract is satisfied.
- Exit `2`: profile/argument/schema error.
- Exit `3`: prerequisite or resource preflight failure.
- Exit `4`: digest, identity, security, or redaction failure.
- Exit `5`: runtime/job failure.
- Exit `6`: evidence finalization/validation failure.
- Exit `7`: unsupported authority request (for example physical-production PASS).

`--json` writes one machine-readable result to stdout. Diagnostics go to stderr. Secret values are never printed.

## Common commands

### `build`

```text
ndnsf-di-deploy build --candidate PATH --variant cpu|gpu --output DIR
```

Produces build artifacts, OCI references, SBOM/provenance, and a release manifest. It must not read deployment identity directories into the build context.

### `verify-release`

```text
ndnsf-di-deploy verify-release --manifest FILE
```

Requires digest-pinned images and verifies manifest, build inputs, SBOM/provenance references, and secret scan result.

### `validate-profile`

```text
ndnsf-di-deploy validate-profile --profile FILE
```

Validates schema and cross-field rules, including adapter exclusivity, GPU resources, storage classes, fallback policy, and multi-node network evidence.

### `preflight`

```text
ndnsf-di-deploy preflight --profile FILE [--output DIR]
```

Runs adapter-specific read-only checks. A failed preflight never starts containers or submits a job.

### `verify-evidence`

```text
ndnsf-di-deploy verify-evidence --evidence FILE
```

Validates schema, digests, promotion manifest, backend truth, redaction, and authority invariants.

### `cleanup`

```text
ndnsf-di-deploy cleanup --profile FILE --dry-run
ndnsf-di-deploy cleanup --profile FILE --apply --older-than DAYS
```

Dry-run is mandatory by default. It never deletes accepted evidence, current/prior release materializations, identities, or referenced models.

## Docker Compose commands

### `install`

```text
ndnsf-di-deploy install --profile FILE
```

Pulls/verifies the OCI digest, renders Compose inputs, validates external identity/state mounts, and records the installed release. It does not imply readiness.

### `start`, `stop`, `status`, `logs`, `evidence`

```text
ndnsf-di-deploy start --profile FILE
ndnsf-di-deploy stop --profile FILE [--timeout SECONDS]
ndnsf-di-deploy status --profile FILE
ndnsf-di-deploy logs --profile FILE [--since TIME]
ndnsf-di-deploy evidence --profile FILE --output DIR
```

`start` waits for declared health checks and NFD route readiness. `stop` preserves declared durable state. `evidence` records resolved image IDs/digests, health, routes, backend, and release lineage.

### `upgrade`, `rollback`

```text
ndnsf-di-deploy upgrade --profile FILE --release-manifest FILE
ndnsf-di-deploy rollback --profile FILE
```

Upgrade is staged and readiness-gated. Rollback uses the stored prior digest/manifest, never a tag.

## Slurm + Apptainer commands

### `materialize`

```text
ndnsf-di-deploy materialize --profile FILE [--force]
```

Converts or selects the profile's pinned OCI artifact, computes SIF SHA-256, records login-node Apptainer version, and refuses checksum mismatch. `--force` creates a new materialization record; it does not overwrite accepted evidence.

### `render-job`

```text
ndnsf-di-deploy render-job --profile FILE --output FILE
```

Renders a deterministic `sbatch` script with explicit partition, account/QOS when used, wall time, CPU, memory, nodes/tasks, and named GPU GRES. It installs evidence finalization traps and contains no secret values.

### `submit`

```text
ndnsf-di-deploy submit --profile FILE [--wait] [--timeout SECONDS]
```

Runs preflight, materializes the SIF, submits exactly one job, returns job ID/run ID, and optionally waits. It must not retry a failed acceptance job automatically.

### `status`, `wait`, `cancel`, `logs`, `evidence`

```text
ndnsf-di-deploy status --job-id ID
ndnsf-di-deploy wait --job-id ID [--timeout SECONDS]
ndnsf-di-deploy cancel --job-id ID [--reason TEXT]
ndnsf-di-deploy logs --job-id ID
ndnsf-di-deploy evidence --job-id ID --output DIR
```

- `status` combines live scheduler state with `sacct` history.
- `wait` uses bounded polling and preserves the terminal scheduler state.
- `cancel` targets only the named job and archives cancellation intent/result.
- `logs` reads declared stdout/stderr paths; it does not tail indefinitely by default.
- `evidence` verifies durable promotion and never converts a failed job to PASS.

## Adapter preflight requirements

### Docker Compose

- Docker and Compose versions;
- release digest availability;
- identity/state mount ownership and read-only secret policy;
- NFD endpoint and required remote ports/routes;
- NVIDIA driver/toolkit only for GPU profiles;
- capacity for image, state, logs, and evidence.

### Slurm + Apptainer

- login node, account/QOS/partition visibility, and requested GRES availability;
- `/project/$USER/ndnsf-di` ownership, capacity/quota signal, and directory policy;
- no bulk artifact target under `/home`;
- Apptainer availability and OCI/SIF identity;
- profile resource limits and job uniqueness;
- multi-node network PASS evidence when `nodes > 1`.

Compute-node preflight repeats Apptainer, actual `/tmp`, allocation, GPU visibility, and driver/runtime observations inside the job.

## Idempotency

- Validation and preflight are read-only.
- `install`/`materialize` are idempotent for the same verified digest/materialization.
- `start` does not create a second Compose project for the same profile.
- `submit` always creates one new historical job/run and therefore requires a unique `runId`.
- `cancel` is safe when a job is already terminal and records that fact.
- Evidence finalization is manifest-based and detects partial promotion.
