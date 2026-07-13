# Spec 109 storage and 0.5B staging verdict

## Verdict

`PASS — the immutable Qwen2.5-0.5B-Instruct source model is sealed in iTiger
project storage; workstation and /home bulk-write sentinels remain clean.`

This verdict authorizes later oracle preparation only. It is not NDNSF-DI
candidate, GPU backend, performance, or physical-production authority.

## Discovery and admission

- Read-only SSH discovery completed with exit 0. Compute-node HTTPS egress is
  available; Apptainer reports `1.3.4-1.el9`; H100, RTX 6000, and RTX 5000 GRES
  classes are visible.
- The administrator-issued initial `/project` allocation is 200 GB. Actual
  `/project/tma1` use before staging was 9,605 bytes. Shared `df` capacity was
  retained but was not treated as the quota.
- The planning envelope admits 0.5B through 32B subject to per-transfer live
  recheck. 72B is `BLOCKED` because its projected 349,868,750,000-byte peak plus
  20 GB reserve exceeds the allocation.

## Transfer and seal

| Operation | Slurm job | Original state/exit | Evidence |
|---|---:|---|---|
| immutable transfer | `146050` | `COMPLETED`, `0:0`, 7m39s | 10 files, 999,604,126 bytes |
| rehash + atomic promotion | `146123` | `COMPLETED`, `0:0`, 9s | sealed registry and transfer manifest |

The accepted source binding is:

```text
repository: Qwen/Qwen2.5-0.5B-Instruct
revision: 7ae557604adf67be50417f59c2c2f167def9a775
registryDigest: sha256:98df31da7cdeaacbfb97ed481782ff1e9eb17961e995b99c6dbb5f30d5f4b2ba
tokenizerDigest: sha256:04e97e50a3e02b4020587defbac67c24cbb31468635b6ae6e30424f950e21a57
licenseDigest: sha256:832dd9e00a68dd83b3c3fb9f5588dad7dcf337a0db50f7d9483f310cd292e92e
model.safetensors: sha256:fdf756fa7fcbe7404d5c60e26bff1a0c8b8aa1f72ced49e7dd0210fe288fb7fe
projectPath: /project/tma1/ndnsf-di/models/source/qwen25-0.5b/7ae557604adf67be50417f59c2c2f167def9a775
```

The original transfer was not retried. Its several-minute `D` state was traced
to `nfs_wait_bit_killable`; the job later completed normally, so the measured
outcome is preserved as-is.

## Safety checks

- `/project/tma1/ndnsf-di/{src,images,models,cache,manifests,evidence}` exists
  under `tma1:users` with protected directory modes.
- The local and `/home` sentinel reports
  `SPEC109_NO_LOCAL_OR_HOME_MODELS_PASS`.
- Cleanup dry-run protects the sealed 0.5B path, current image, and all evidence;
  only explicit unreferenced examples appear in its delete set.
- Physical production remains `DEFERRED`, owner Spec 106.
