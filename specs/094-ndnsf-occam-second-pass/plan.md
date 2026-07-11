# Implementation Plan: NDNSF Occam Second Pass

## Constitution Check

- Use CodeGraph before edits and exact searches after semantic exploration.
- Preserve V2 invocation, security, fail-closed behavior, and MiniNDN-first
  validation.
- Treat negative tests and failed experiments as evidence.
- Keep every removal independently reviewable and revertible.
- Do not modify proposal slides or papers.

## Unified Design

### 1. Canonical DI Load Driver

`threaded` owns measured open-loop offered load because it reuses initialized
ServiceUser instances and passed the scheduling gates through 8 RPS. `child`
remains an explicit diagnostic isolation mode. Remove `process-pool`, its
five-second schedule lead, worker-index batch protocol, helper metrics, choices,
defaults, GUI values, and tests.

The old standalone RPS sweep is removed instead of repaired. It hardcodes the
deterministic runner and defines stability using only return code and success
rate. The canonical recipe is the NativeTracer MiniNDN harness with explicit
60-second duration, request cap, threaded mode, and Spec 093 gates.

### 2. Canonical DI Operator/API Surface

The version-2 `ThreeRoleGuiProfile` and direct role tabs are the sole role
configuration/runtime surface. Remove the old profile model and duplicate
Script Controller/User/Provider tabs. Supporting Project Wizard, Policy Editor,
Model Split, Certificates, Qwen MiniNDN, and regression runner remain because
they provide distinct operations rather than a second role runtime.

`artifact_references` is the sole DI artifact input. Remove the unused
`repo_manifests` keyword and helper that arbitrates between the two names.
Internal helpers are renamed to artifact terminology.

### 3. Canonical Repo Storage And Request Surface

SQLite is authoritative; tiered memory is a bounded cache. Remove the public
in-memory authoritative store and default memory-backed constructors. Production
and examples construct a tiered SQLite store explicitly. Cache tests use SQLite
or a test-local fault-injection backend.

Remove ignored Repo options instead of pretending they alter behavior:
`producer_retention_s` is obsolete after the always-on data plane, and
`isolated_runtime` is unsafe because a second ServiceUser with the same identity
and SVS session is not created. All callers use the canonical shared runtime.

### 4. Typed Status And Accurate Audit

Keep one typed `ServiceOperationStatus`; remove nested `legacyStatus` copies.
Repo-specific capability fields remain a service payload and are renamed
locally to avoid false legacy classification.

Replace broad text-only Occam patterns with path-aware prohibited-mechanism
rules. Correctly owned DI artifacts, internal Repo bindings, abstract methods,
optional ACK handlers, and typed operation status are not violations.

## Migration And Rollback

- Process-pool users select `threaded`; diagnostic isolation uses `child`.
- Old GUI profiles are converted once to version 2 before upgrade; no automatic
  migration shim remains in runtime code.
- `repo_manifests=` callers rename the keyword to `artifact_references=`; exact
  inventory currently finds none outside definitions.
- Repo constructors receive `makeTieredRepoStore(path, budget)` explicitly.
- Removed ignored arguments are deleted from callers with no behavior change.
- Each implementation phase is a separate commit and can be reverted without
  reverting the other phases.

## Validation Strategy

1. Static prohibited-symbol and caller scans.
2. Focused Python GUI/runtime/DI/Repo tests.
3. C++ Core, Repo, DI, and UAV build/focused test targets.
4. Full Python suite.
5. 60-second threaded NativeTracer MiniNDN validation plus Repo/UAV/Core quick
   network regressions selected by the existing quick-check runner.
6. Post-implementation Spec Kit analyze/audit/converge and GSD health.

## Stop Conditions

- A supposedly dead symbol has an active external/internal caller.
- A removal weakens security, persistence, fail-closed behavior, or typed wire
  semantics.
- Disk falls below 3 GiB or a stale MiniNDN process contaminates evidence.
- A baseline regression cannot be distinguished from the proposed edit.
