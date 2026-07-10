# Experiment Plan

## Baselines

- Spec 077 c16/2 RPS: write mean 23,407 ms; overall p95 28,127 ms; 119/120.
- Spec 077 write-heavy c4/0.5 RPS: write mean 5,793 ms; p95 10,583 ms; 28/30.

## Controls

Use the same MiniNDN topology, three Repo identities, RF=2, W=ALL, 2,048-byte
objects, seeds, NFD/NLSR setup, cache size, and 60-second measured windows.

## Measures

- read/write p50, p95, p99, mean, and failure rate;
- reserve and store phase latency;
- normal/Targeted/async/fallback/timeout counters;
- valid receipts and incomplete writes;
- maximum concurrent replica operations;
- offered and achieved RPS.

## Interpretation

Improvement requires correct receipts plus lower write p95. A lower mean with
worse failure rate or a hidden longer timeout is not an improvement. Report
token bootstrap/refill spikes separately from steady Targeted fast-path calls.
