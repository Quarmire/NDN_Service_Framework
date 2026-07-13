# Large-model capacity and placement gate

## Live refresh

Read-only iTiger discovery completed with SSH exit 0. The cluster exposed eight
H100 80 GB, RTX 6000 48 GB, and RTX 5000 32 GB devices per applicable node;
Apptainer remained 1.3.4. Project shared free space was about 927 TB and `/tmp`
free space about 736 GB. Shared filesystem capacity is not treated as the user
quota.

The only allocation authority remains the administrator-issued 200 GB initial
`/project` quota. The cluster discovery command does not expose a verified
per-user byte quota, so the record explicitly says `commandVerified=false`.

## Mechanical decisions

- **32B: BLOCKED.** Its planning envelope plus reserve fits inside 200 GB, but
  Spec 109 requires a sealed file manifest rather than a parameter estimate for
  large-model admission. The exact Spec 107/108 predecessor gate is also
  blocked. No transfer or job started.
- **72B: BLOCKED.** The 349,868,750,000-byte planning peak plus 20 GB reserve
  exceeds 200 GB before any transfer. A quota of at least 370 GB, plus any
  administrator-required filesystem margin, must be approved before a sealed
  file-manifest admission can be attempted. No transfer or job started.
- **Multi-node: DEFERRED.** Spec 108 T134 is incomplete, so no multi-node
  candidate identity exists. One-node multi-GPU remains the first admissible
  placement after the predecessor and storage gates pass.

The live raw/parsed discovery and decisions are under
`results/spec109-itiger-qwen/large-model-admission/`. All 32B/72B matrix cells
remain terminal and visible; physical-production authority stays with Spec 106.
