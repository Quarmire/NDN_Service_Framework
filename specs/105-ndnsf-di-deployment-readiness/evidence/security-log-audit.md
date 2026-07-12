# T097 Secret and payload leakage audit

Status: **BLOCK for retained historical INFO evidence; source/package
remediated**.

## Scope

- all files under `specs/105-ndnsf-di-deployment-readiness/evidence/`;
- the three retained T062 result directories and their INFO logs/CSV/JSONL;
- `packaging/ndnsf-di-systemd/`;
- a freshly generated digest-bound release at
  `/tmp/spec105-security-scan-release`.

## Negative scans

The following classes returned no matches (ripgrep exit 1):

- PEM/OpenSSH private-key headers and Bearer authorization values;
- assigned password, API key, private key, access-token or refresh-token values;
- secret/password assignments in unit, environment, JSON and configuration
  files;
- private material in the generated release bundle.

Systemd examples contain paths only. Deployment doctor output redacts `device`
and every `secret_files` entry. Metrics snapshots expose numeric counters,
gauges and operator labels; they have no request payload field.

## Finding and remediation

One retained T062 INFO log contains the full public 32-token correctness oracle
in `LLM_PIPELINE_OPEN_LOOP_SUMMARY expectedTokens=[...]`. It is not a secret,
key or user-private prompt, and the same frozen oracle is preregistered in
`qwen-scheduler-revision.md`, but it is still payload-derived token content and
therefore violates the strict zero-payload logging rule.

The source is fixed to emit only `expectedTokenCount` and a SHA-256
`expectedTokenDigest`; `TOKEN_MISMATCH` now reports only the mismatch index.
The deployment-readiness test asserts the old `expectedTokens=` field is absent.
Focused Python tests pass 19/19.

The historical T062 log is immutable measurement evidence and was not edited or
replaced. Because rerunning T062 would violate its no-fourth-run rule, no new
live Qwen log can prove the remediation in this feature. The application
security dimension therefore remains BLOCK rather than claiming zero leakage.
No secret exposure was found, and future candidate logs use the remediated
format.
