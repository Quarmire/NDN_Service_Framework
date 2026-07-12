# Security Design

## Preserved Invariants

- ServiceController-signed, target-certificate-encrypted permissions.
- REQUEST/SELECTION `/SERVICE/<service>` and ACK/RESPONSE
  `/PERMISSION/<service>` NAC-ABE routing.
- One-time UserToken and ProviderToken proofs with replay rejection.
- Provider permission before service readiness.
- Fail-closed generic admission and DI execution leases.
- Digest verification for plans, artifacts, dependencies, and executable bundles.
- Existing allowlist plus sandbox requirements for executable artifacts.

## New Security Bindings

- Execution evidence binds authenticated provider, boot, runner, device, plan,
  model and artifact identity.
- Telemetry is advisory and authenticated but never grants execution authority.
- Plan feasibility cannot override provider admission or a missing lease.
- KV state binds service, provider boot, stage, model, plan, session, context and
  security epoch.
- Attempt epochs prevent a late old attempt from becoming the accepted result.
- Cache files contain no authority; deletion or rollback is always safe.

## Secrets and Host Permissions

- systemd services run under dedicated least-privilege users;
- identity/key references live in root/operator-managed paths with restrictive
  mode and are not copied into profiles, logs, metrics or release bundles;
- provider artifact cache, runtime state and metrics have distinct writable
  directories;
- services receive only required NFD sockets/devices and no blanket root shell;
- the deployment doctor reports permissions and identity names, never key bytes.

## Required Negative Tests

- forged/missing execution evidence;
- evidence identity/digest mismatch;
- telemetry replay and stale sequence;
- plan signed for another provider/model/artifact;
- stale attempt output and replayed execution lease;
- KV reference across provider boot, model, plan, session or security epoch;
- executable artifact missing trust anchor, allowlist, sandbox or digest;
- rollback with incompatible cache metadata;
- real security path under measured load.

## Explicitly Forbidden

- `ValidatorNull`, forced authorization, disabled tokens, plaintext permission
  response, or unsigned executable trust in any release gate;
- presenting MiniNDN dummy-keychain execution as cryptographic-strength or
  physical-production security evidence;
- logging prompt, tensor, token, key, provider-token or KV payload bytes;
- treating resource telemetry as an authorization decision.
