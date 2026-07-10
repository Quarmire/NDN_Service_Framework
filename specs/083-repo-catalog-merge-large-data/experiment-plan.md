# Experiment Plan

## Material Passport

- Experiment ID: `repo-catalog-merge-large-data-083`
- Type: deterministic MiniNDN code/network experiment
- Verification status: planned

## Variables

- Independent variable: 16 inline merge batches per initial peer delta versus
  one segmented pull reference.
- Fixed: seed 78004, workers=3, max-jobs=6, 60 seconds, 0.5 RPS, concurrency 4,
  RF=3/W=QUORUM, 2 KiB objects, failure at 20 seconds, restart after 12 seconds.
- Outcomes: merge mode/batches/duration, request p50/p95, first repair, repair
  coverage, W floor, and invalid repairs.

## Rules

- Correctness gates dominate latency.
- Run the treatment once; no favorable-result reruns.
- Report the next observed bottleneck rather than extending scope mid-run.
