# Deferred Physical Pilot Quickstart

Do not execute until the entry gate in `plan.md` passes.

```bash
ndnsf-di inspect <spec105-release-gate.json>
ndnsf-di doctor --profile <physical-cluster.json> --json
sudo packaging/ndnsf-di-systemd/install.sh --release <candidate-release>
ndnsf-di bench --campaign <spec106-canary.json> --out <unique-dir>
ndnsf-di bench --campaign <spec106-soak.json> --out <unique-dir>
```

Expected: real identities and device-bound CUDA evidence; two clean installs;
matched canary; bounded restart/rollback; one uninterrupted 24-hour evidence
record. Any absent prerequisite or cell yields production BLOCK.
