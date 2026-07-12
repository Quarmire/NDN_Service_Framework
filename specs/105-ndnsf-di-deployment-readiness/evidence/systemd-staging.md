# T091 Isolated systemd staging evidence

Status: **PASS** for local packaging/staging. This is not a physical systemd
deployment and does not change the Spec 106 `DEFERRED` status.

## Environment

- Date: 2026-07-12
- Host systemd: `245 (245.4-4ubuntu3.24)`
- Source branch: `Experimental`
- Validation script: `packaging/ndnsf-di-systemd/validate-staging.sh`
- Passing work root: `/tmp/spec105-systemd-staging-20260712T120500Z`
- Backend scope: packaging-only stubs; no compute claim

## Frozen command

```bash
packaging/ndnsf-di-systemd/validate-staging.sh \
  --work-root /tmp/spec105-systemd-staging-20260712T120500Z
```

## Results

- Generated release N (`spec105-staging-n`) and N+1
  (`spec105-staging-n1`) with source commit, schema compatibility and SHA-256
  manifests.
- `sha256sum -c SHA256SUMS`: PASS for every file before each activation.
- First N install: PASS.
- Repeated N install: PASS and idempotent.
- N -> N+1 activation: PASS.
- N+1 -> N rollback: PASS; `current` resolved to
  `releases/spec105-staging-n` and `previous` retained N+1.
- Authoritative Repo sentinel digest before/after:
  `75a0a93ec792ce49960c0ef009e2fffd238a1c97d4ca3c29d2f12ca996c7542b`;
  unchanged.
- `systemd-analyze verify` on isolated unit copies with only `ExecStart`
  replaced by `/bin/true`: PASS. Ordering and requirements used the included
  isolated `nfd.service` stub.
- Static security check: PASS for 9 required directives:
  `NoNewPrivileges`, `PrivateTmp`, strict system protection, home/kernel/control
  group protection, SUID/SGID restriction, bounded restart and stop timeout.
- No host service was started, stopped, enabled or reloaded.

## Retained failed attempt and limitation

The first work root, `/tmp/spec105-systemd-staging-20260712T120000Z`, completed
all release/install/rollback/Repo checks but stopped because this systemd 245
build reports `Option --root is only supported for cat-config right now` for
`systemd-analyze verify --root`. The passing script therefore verifies temporary
unit copies through an isolated `SYSTEMD_UNIT_PATH`; it changes `ExecStart` only
to avoid resolving `/opt` in the host namespace. This proves package syntax,
dependency graph and static hardening, not live PID-1 supervision. Physical
supervisor behavior remains Spec 106 work.
