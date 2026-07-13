# Quickstart: NDNSF-DI OCI Deployment Adapters

> Design-time command contract. Paths and commands become executable as the tasks in `tasks.md` are implemented.

## 1. Build and verify one OCI release

```bash
cd /home/tianxing/NDN/ndn-service-framework
packaging/ndnsf-di-container/bin/ndnsf-di-deploy build \
  --candidate specs/107-ndnsf-di-minindn-perf-fault-candidate \
  --variant cpu \
  --output dist/ndnsf-di-container

packaging/ndnsf-di-container/bin/ndnsf-di-deploy verify-release \
  --manifest dist/ndnsf-di-container/release-manifest.json
```

For GPU builds, use `--variant gpu` and verify the declared CUDA/ONNX Runtime compatibility. Never add a host NVIDIA kernel driver to the image.

## 2. Cloud Docker Compose adapter

### Prerequisites

- supported Linux host;
- Docker Engine and Compose plugin;
- TCP/UDP 6363 between hosts for multi-host NFD;
- for GPU: compatible NVIDIA driver and NVIDIA Container Toolkit;
- node-specific identity and secret files outside the OCI image.

### Validate and start a CPU node

```bash
export NDNSF_PROFILE=/etc/ndnsf-di/profiles/cloud-node-a.yaml
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy preflight \
  --profile "$NDNSF_PROFILE"
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy install \
  --profile "$NDNSF_PROFILE"
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy start \
  --profile "$NDNSF_PROFILE"
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy status \
  --profile "$NDNSF_PROFILE"
```

### Collect evidence and operate

```bash
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy evidence \
  --profile "$NDNSF_PROFILE" --output /var/lib/ndnsf-di/evidence
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy logs \
  --profile "$NDNSF_PROFILE"
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy upgrade \
  --profile "$NDNSF_PROFILE" --release-manifest /path/to/new-release.json
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy rollback \
  --profile "$NDNSF_PROFILE"
```

The profile selects `runtime.kind: docker-compose`. One host-scoped NFD is the default. Multi-host profiles must declare routes and pass reachability preflight.

## 3. iTiger Slurm + Apptainer adapter

### Prerequisites

1. Connect to the University of Memphis VPN.
2. Verify `ssh itiger` works.
3. Do not run NDNSF-DI or NFD persistently on the login node.
4. Keep bulk artifacts out of `/home`.

### Create the durable layout

```bash
ssh itiger 'mkdir -p \
  /project/$USER/ndnsf-di/{src,releases,sif,models,identities,evidence,logs}'
```

Default layout:

```text
/home/$USER/                         SSH and small config only
/project/$USER/ndnsf-di/src/         source or deployment bundle
/project/$USER/ndnsf-di/releases/    OCI release manifests/provenance
/project/$USER/ndnsf-di/sif/         verified SIF materializations
/project/$USER/ndnsf-di/models/      durable models
/project/$USER/ndnsf-di/identities/  external per-job/node identity bindings
/project/$USER/ndnsf-di/evidence/    durable run evidence
/tmp/ndnsf-di-$SLURM_JOB_ID-$RUN_ID compute-node job scratch
```

### Preflight from the login node

```bash
ssh itiger 'cd /project/$USER/ndnsf-di/src && \
  packaging/ndnsf-di-container/bin/ndnsf-di-deploy preflight \
    --profile packaging/ndnsf-di-container/adapters/slurm-apptainer/profiles/itiger-rtx5000.yaml'
```

Preflight checks account/partition/GRES visibility, project paths and quota information, profile consistency, OCI digest, and Apptainer availability. GPU, scratch, and execution versions are checked again inside the allocation.

### Submit the initial five-minute RTX 5000 acceptance job

```bash
ssh itiger 'cd /project/$USER/ndnsf-di/src && \
  packaging/ndnsf-di-container/bin/ndnsf-di-deploy submit \
    --profile packaging/ndnsf-di-container/adapters/slurm-apptainer/profiles/itiger-rtx5000.yaml \
    --wait'
```

The rendered request must be equivalent to:

```text
--partition=bigTiger
--gres=gpu:rtx_5000:1
--cpus-per-task=2
--mem=8G
--time=00:05:00
```

Do not hard-code a physical GPU index. The job records Slurm allocation data, host GPU UUID/model, `CUDA_VISIBLE_DEVICES`, and the GPU seen through `apptainer exec --nv`.

### Inspect, cancel, and validate evidence

```bash
ssh itiger 'cd /project/$USER/ndnsf-di/src && \
  packaging/ndnsf-di-container/bin/ndnsf-di-deploy status --job-id JOB_ID'
ssh itiger 'cd /project/$USER/ndnsf-di/src && \
  packaging/ndnsf-di-container/bin/ndnsf-di-deploy cancel --job-id JOB_ID'
ssh itiger 'cd /project/$USER/ndnsf-di/src && \
  packaging/ndnsf-di-container/bin/ndnsf-di-deploy verify-evidence \
    --evidence /project/$USER/ndnsf-di/evidence/RUN_ID/evidence.json'
```

Successful substrate acceptance requires:

- terminal Slurm state `COMPLETED` and exit code `0:0`;
- requested and observed GPU allocation;
- host and Apptainer GPU observations;
- pinned OCI digest and SIF SHA-256;
- compute-node `/tmp` bounded write/fsync result;
- checksummed evidence promotion to `/project`;
- redaction and schema validation.

Candidate GPU acceptance additionally requires actual NDNSF-DI ONNX Runtime inference and observed provider evidence. Physical-production remains `DEFERRED` for Spec 106.

## 4. Other GPU classes

Parameterize the same profile and acceptance flow with:

```text
gpu:h100_80gb:1
gpu:rtx_6000:1
gpu:rtx_5000:1
```

Always re-run partition/GRES discovery. Cluster node labels, versions, quotas, and availability are external mutable facts.

## 5. Multi-node iTiger gate

Do not enable multi-node NDNSF-DI merely because a multi-node allocation succeeds. First run the dedicated in-allocation network probe and retain:

- node addresses selected for NFD;
- bidirectional TCP and UDP 6363 results;
- NFD face creation and route results;
- scheduler job/node mapping;
- firewall or policy failure evidence.

Until it passes, the profile validator rejects `nodes > 1` unless `network.preflightEvidence` references an admissible PASS bundle.

## 6. Cleanup

```bash
# Compose: stops runtime but preserves declared durable state.
sudo packaging/ndnsf-di-container/bin/ndnsf-di-deploy stop --profile "$NDNSF_PROFILE"

# Slurm: cancel only the named job; never use broad user-wide cancellation by default.
ssh itiger 'scancel JOB_ID'
```

Remove per-job `/tmp` scratch automatically. Retain only accepted/canonical SIF and evidence in `/project`; prune superseded caches through an explicit, dry-run-capable cleanup command.
