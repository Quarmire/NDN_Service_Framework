# T004 — Operator and Packaging Surface Inventory

## Existing Maintained Surfaces

| Surface | Current state |
|---|---|
| `tools/ndnsf_runtime.py` | stdlib profile validation, resolved output, runtime doctor, DI launch/campaign/sweep/search wrappers |
| `runtime_v1.py provider` | emits sample/runtime ACK metadata rather than running the production provider |
| `runtime_v1.py plan` | builds a local plan lease |
| `runtime_v1.py run` | executes `simulate_prefill_decode` and reports `executed-contract-smoke` |
| `runtime_v1.py bench/context-sweep` | generates local simulated summaries |
| `runtime_v1.py inspect/schema-sample` | reads/writes contract JSON fixtures |
| `examples/DI_NativeProviderExecutable.cpp` | real provider process entrypoint used by MiniNDN harnesses |
| MiniNDN experiment scripts | current authoritative end-to-end launch and evidence collection paths |

## Missing Production-Candidate Surfaces

- no `packaging/ndnsf-di-systemd/` tree;
- no DI install/activate/rollback/uninstall scripts;
- no versioned DI release manifest;
- no supported `status` or `metrics` operator command;
- no atomic Prometheus textfile/JSON metrics exporter;
- no systemd unit hardening, tmpfiles or logrotate configuration;
- no local clean-staging canary/upgrade/rollback/24-hour runbook;
- no six-dimension candidate release-gate generator.

## Existing Reusable Patterns

- `packaging/uav-release/` demonstrates versioned release construction and
  documentation but is UAV-specific and must not become DI policy.
- Core/Repo authoritative state, Targeted security and executable-artifact
  allowlist/sandbox contracts remain reusable invariants.
- Current MiniNDN scripts remain the algorithm validation substrate; production
  CLI adapters should call the same runtime rather than duplicate it.

## Ownership Decision

DI owns model/runtime profiles, operator commands and systemd packaging. Core
continues to own generic dynamic invocation, permission/token/replay and
execution-lease mechanisms. Spec 105 validates local packaging; Spec 106 alone
owns physical profiles and production release authority.
