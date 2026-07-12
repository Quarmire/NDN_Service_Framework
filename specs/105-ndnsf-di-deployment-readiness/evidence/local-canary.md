# T092 Local MiniNDN canary preflight

Status: **NOT RUN / BLOCK**.

The two prespecified clean records were created by:

```bash
python3 Experiments/spec105_local_canary_preflight.py \
  --out results/spec105-local-canary-1-20260712T122000Z \
  --record-id spec105-local-canary-1-20260712T122000Z
python3 Experiments/spec105_local_canary_preflight.py \
  --out results/spec105-local-canary-2-20260712T122000Z \
  --record-id spec105-local-canary-2-20260712T122000Z
```

Both commands exited 2 and wrote immutable `preflight.json`. Each records the
host (`tianxing-VirtualBox`, Linux 5.15.0-139, x86_64, 4 CPUs, 12.54 GB RAM),
local `onnxruntime-cpu` backend, no physical GPU evidence, campaign digest
`15e4afa35c5e1896c01d09264c34127cc0cbec7ec4c42c7d1b9e488308d7f26f`,
the intended matched single-node/three-provider cells, 60-second/1 RPS profile,
source commit and available disk.

The controlling immutable T062 result is `BLOCK`: 25/60 complete (41.6667%),
0.4167 achieved RPS and 20.17x p95 ratio. Therefore no duplicate live Qwen cell
was executed. This is required by the sequential hard gate and prevents an
adaptive replacement campaign after failure. Candidate operations remain
`BLOCK`; physical status remains `DEFERRED` to Spec 106.
