# Spec 108 Capability Gap at Spec 110 Start

Snapshot date: 2026-07-13. Spec 108 has 83 unchecked tasks. It already provides
the CPU OCI source, Compose foundation, Slurm/Apptainer substrate adapter, GRES
profiles, storage checks, job traps, and offline adapter tests. That is not yet
a GPU NDNSF-DI release.

## Blocking runtime and deployment gaps

| Tasks | Missing capability | Source owner |
|---|---|---|
| T091-T103 | Pinned GPU dependencies, compatibility matrix, GPU OCI, `--nv`, backend/UUID correlation, fail-closed fallback, real RTX 5000 inference | `packaging/ndnsf-di-container/oci/`, `lib/adapters/`, `lib/evidence.py` |
| T104-T114 | Runtime identity binds, artifact/evidence secret scanning, protected cleanup | `lib/profile.py`, adapters, OCI scanner, `lib/cleanup.py` |
| T115-T126 | Slurm lifecycle, exact-job cancellation, logs, rollback/recovery | `lib/release.py`, `slurm_apptainer.py`, deploy CLI |
| T127-T135 | Adapter conformance and measured multi-node NFD network gate | contract tests, `probe-multinode-network.sh`, profile validation |
| T136-T160 | Full acceptance, documentation, traceability, audit, Spec 106 handoff | tests, docs, evidence, Spec Kit artifacts |

The older Slurm submission path writes `submission.json` only after `sbatch`
returns and the Qwen template does not yet start the complete NFD/controller/
user/three-provider process graph. Spec 110 therefore owns the crash-safe
pre-submit journal and full allocation runner instead of treating Spec 108 T087
or a `/bin/true` substrate probe as candidate inference.

Spec 110 can reuse valid Spec 108 mechanisms by digest. It must not claim that
Spec 108 T090/T103 or its final acceptance tasks passed unless their original
task contracts are independently executed and evidenced.
