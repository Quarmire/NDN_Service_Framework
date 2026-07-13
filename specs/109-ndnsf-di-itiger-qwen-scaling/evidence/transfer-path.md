# iTiger Qwen transfer path

## Verdict

`PASS — direct compute-node HTTPS egress is available; model bytes are staged by
a bounded CPU Slurm job directly into project quarantine.`

## Observation

The read-only discovery executed on 2026-07-13 through
`tma1@itiger.memphis.edu` reported:

```text
NDNSF_DISCOVERY|EGRESS|PASS
NDNSF_DISCOVERY|APPTAINER|1.3.4-1.el9
```

The exact local mirror is
`results/spec109-itiger-qwen/discovery/raw.stdout`. The result is substrate and
transfer-path authority only; it is not model, candidate, performance, or
physical-production evidence.

## Selected path

1. Resolve and lock the public Qwen repository to a 40-hex commit through the
   official Hugging Face API; never accept `main` as an execution revision.
2. Submit one CPU-only Slurm transfer job with bounded time, CPU, and memory.
3. Download on the compute node into
   `/project/$USER/ndnsf-di/models/.partial/<model>-<revision>`.
4. Reject symlinks and unresolved Git LFS pointer text, hash every materialized
   file, and write a transfer manifest under the project manifest directory.
5. Validate the full file set and atomically rename quarantine into the sealed
   content path. No model bytes transit or persist on the workstation or in
   `/home`.

No administrator fallback is currently needed. If direct egress later fails,
the cell must become terminally blocked until an administrator-approved project
transfer mechanism is documented; the operator must not silently download the
model locally.
