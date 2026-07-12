# Operator CLI Contract

Supported production-facing commands:

```text
ndnsf-di doctor --profile <deployment.json> [--json]
ndnsf-di provider --profile <provider.json>
ndnsf-di plan --model <model.json> --providers <cluster.json> --out <plan.json> --explain <explain.json>
ndnsf-di run --profile <deployment.json> --plan <plan.json> --request <request.json> --out <result.json>
ndnsf-di status --profile <deployment.json> [--json]
ndnsf-di metrics --profile <deployment.json> --format json|prometheus-textfile --out <path>
ndnsf-di bench --campaign <campaign.json> --out <unique-dir>
ndnsf-di inspect <result-or-gate.json>
ndnsf-di contract-smoke ...
```

Rules:

- `provider`, `run`, and `bench` execute real paths; simulated commands move to
  `contract-smoke`.
- `--json` output has stable schema and nonzero exit on unhealthy/BLOCK.
- commands print resolved profile, release, plan and evidence identities.
- secrets are referenced by protected paths and never printed.
- destructive cache cleanup requires an explicit disposable-cache scope; Repo
  authoritative state is never included.
