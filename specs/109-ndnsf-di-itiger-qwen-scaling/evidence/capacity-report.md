# Qwen storage and GPU capacity report

## Durable storage

- Administrator allocation authority: 200 GB `/project`.
- Sealed source retained: Qwen2.5-0.5B-Instruct, 999,604,126 bytes across ten
  files at an immutable revision.
- Workstation and `/home` bulk-model writes: zero by sentinel.
- Latest shared `/project` free capacity: about 927 TB; this is not interpreted
  as the user quota.
- Latest compute `/tmp` free capacity: about 736 GB; scratch is temporary and
  cannot establish durable admission.

## Size decisions

| Size | Storage decision | GPU/placement decision | Execution |
|---|---|---|---|
| 0.5B | source SEALED | predecessor blocked | zero candidate jobs |
| 1.5B–14B | planning envelope fits | predecessor blocked | zero transfers/jobs |
| 32B | estimate fits, sealed file manifest absent | predecessor blocked | zero transfers/jobs |
| 72B | at least ~370 GB including reserve required | not admitted | zero transfers/jobs |

The refreshed GRES inventory exposes H100 80 GB, RTX 6000 48 GB, and RTX 5000
32 GB nodes. One-node multi-GPU is the preferred large-model placement. A
multi-node variant remains deferred until Spec 108 T134 supplies admissible
network evidence.

These are admission and inventory results, not measured model capacity or
physical-production readiness.
