# iTiger runtime release contract

## Build boundary

The release source is an OCI image referenced by digest. It may be built by CI,
a remote Docker-capable builder, or another controlled OCI builder. iTiger does
not require or run a Docker daemon.

## Required SIF contents

- NFD and ndn-cxx programs/libraries;
- ndn-svs/NAC-ABE dependencies;
- NDNSF C++ runtime and Python binding;
- NDNSF-DI Python/C++ runtime and launchers;
- PyTorch and Transformers/tokenizer tooling;
- ONNX Runtime GPU and compatible CUDA user-space libraries;
- Qwen oracle/export/stage-provider commands;
- evidence and compatibility probes.

## Host/runtime split

| Host provides | SIF provides |
|---|---|
| Slurm allocation | all project applications/libraries |
| Apptainer executable | exact Python and C++ runtime |
| NVIDIA kernel driver/devices | CUDA user-space, PyTorch, ORT GPU |
| project filesystem and compute `/tmp` | entrypoints/config templates |

Invocation is `apptainer exec --nv` with a clean environment and explicit bind
allowlist. NVIDIA Container Toolkit is neither required nor installed on iTiger.

## Forbidden image content

Private keys, VPN/SSH credentials, MFA artifacts, registry/model access tokens,
user home directories, mutable model weights, accepted evidence, and secrets.

## Acceptance

A release is `PASS` only when OCI digest, SIF checksum, dependency locks, secret
scan, compute-node imports/linking, NFD lifecycle, allocated GPU correlation,
PyTorch CUDA operation, and ONNX Runtime CUDA provider execution all pass.

## Rollback

The current and prior accepted OCI/SIF digests remain protected. Rollback means
freezing a new run/candidate binding to the prior accepted release; no mutable
`current.sif` replacement may alter an already frozen candidate. Failed SIF
materialization removes only its partial path and never the prior release.
