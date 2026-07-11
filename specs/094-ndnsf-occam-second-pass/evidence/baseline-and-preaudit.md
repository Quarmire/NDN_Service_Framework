# Baseline And Pre-Implementation Audit

## Baseline

- Commit before implementation: `74015ed`.
- CodeGraph: up to date, 2,150 files, 47,552 nodes, 159,243 edges.
- Disk: 8.1 GiB available on the workspace filesystem.
- Python regression:

  ```bash
  PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper \
    python3 -m unittest discover -s tests/python -p 'test_*.py'
  ```

  Result: 344 passed, 1 skipped in 10.083 seconds.

- The host Python does not provide pytest. An initial pytest/module-name
  invocation executed no tests and was replaced by the repository's supported
  unittest discovery command.
- Current Occam inventory: 178 textual findings, of which 47 are classified as
  active by the old broad scanner. Code-aware review found that most are false
  positives for app-owned classes, abstract methods, optional ACK handlers, and
  typed status.

## Pre-Implementation Gate

**Verdict**: PASS

The strict structure audit initially blocked because tasks lacked `[US#]`
labels and traceability. After adding both, it reports:

```text
functional_requirements: 13
success_criteria: 6
user_stories: 5
tasks: 27
missing_story_tasks: []
has_traceability: True
traced_requirements: 13
```

Code reality supports each REMOVE decision:

- process-pool is a public option but previously failed the scheduling gate;
- the old sweep has one dedicated test caller and incomplete stability logic;
- old GUI profile/tabs duplicate the current role tabs;
- `repo_manifests` has no caller outside its own definitions;
- memory-only Repo is reached only through convenience constructors/tests;
- `producer_retention_s` and `isolated_runtime` are explicitly discarded;
- no consumer reads nested `legacyStatus`.

No deletion changes V2 invocation, NAC-ABE, tokens, permission encryption,
typed ACK writers, fail-closed leases, SQLite persistence, replication/repair,
stream semantics, UAV codec/FEC/ROI policy, or DI runtime/planning/cache state.
The mixed ACK reader and internal Repo native binding remain behind their
existing 2026-12-31/major-release review gates.

## Adversarial Review

- **Strongest counterargument**: experimental public APIs may have external
  callers not visible in this checkout. Mitigation: every removed input fails
  visibly, migration is one-line, and each phase is independently revertible.
- **Persistence challenge**: removing memory-only could make tests slower.
  This is acceptable because memory-only authority contradicts the product
  contract; test-local fakes remain allowed for fault injection.
- **Measurement challenge**: deleting a sweep could reduce convenience. The
  old convenience produces a misleading stable-RPS claim; direct harness
  recipes are preferable until one strict canonical sweep is designed.
- **Security challenge**: compatibility readers might appear redundant. They
  are explicitly retained because their bounded migration contract has not
  expired.
