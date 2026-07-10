# Experiment Plan

## Questions

1. Does confirmed-write logic prevent false replication success during node loss?
2. Does the long-lived data plane keep producer/thread count bounded as concurrency rises?
3. Do concurrent reads scale better than the serialized baseline without correctness loss?
4. How quickly do reads fail over and replicas repair after one Repo disappears?

## Independent Variables

- concurrency: 1, 4, 16, 32 clients;
- workload: 90/10 read/write, 50/50 mixed, 10/90 read/write;
- replication factor: 1, 2, 3;
- object form: opaque segmented object and exact signed packet set;
- failure phase: none, mid-write, mid-read, pre-repair, mid-repair;
- implementation mode: legacy producer path only where retained as a diagnostic baseline, and always-on data plane.

## Controls

- same MiniNDN topology, object sizes, names, random seed, security policy, cache budget, SQLite location, NFD/NLSR configuration, and 60-second measured window;
- warmup completes before measured requests;
- timeline/log sampling remains fixed;
- foreground and repair limits are recorded.

## Measures

- attempted/completed/failed/rejected operations;
- stable RPS and offered RPS;
- p50/p95/p99 end-to-end latency;
- valid receipt count and false-commit count;
- failover and repair latency;
- cache hit ratio and SQLite read/write latency;
- queue/inflight maxima, producer/thread count, reserved/used bytes;
- packet name/wire digest and object digest correctness.

## Procedure

1. Run focused unit and in-process concurrency tests.
2. Run MiniNDN smoke with three Repos and replication factor three.
3. Run each workload for 10 seconds warmup plus 60 seconds measurement.
4. Repeat each stable-RPS candidate three times; use deterministic seeds.
5. Inject one controlled Repo termination after the measurement phase begins.
6. Restart the Repo where required and measure catalog/repair convergence.
7. Preserve one canonical machine-readable result directory per scenario.

## Acceptance

- zero false commits and zero wire/name corruption;
- bounded producer/thread count independent of request count;
- no capacity oversubscription;
- successful failover within configured deadline when a live replica exists;
- repair restores configured replication within the campaign repair deadline;
- stable-RPS result reports all required metrics, including negative results.

## Accepted Evidence (2026-07-10)

- Build and contracts: full `./waf build`; four Repo C++ executables passed;
  57 focused Python Repo tests passed.
- Canonical aggregate: 9 campaigns in
  `results/repo_ha_spec077_canonical_20260710/`.
- Read c1, 0.5 RPS: 30/30, p50 135.626 ms, p95 655.432 ms.
- Read c4, 1 RPS: 59/60, p95 1,144.134 ms.
- Read c16, 2 RPS: 119/120, but p95 28,127.155 ms. The aggregate's
  `stable` flag means failure rate at most 1%; it is not a tail-latency SLO.
- Read c32, 4 RPS: 202/240, 15.83% failure and 38 admission rejections.
- Mixed c4, 0.5 RPS: 40% failure; write-heavy c4, 0.5 RPS: 6.67% failure.
  Confirmed replicated writes remain correct, but the serialized NDNSF/SVS
  control plane is the measured throughput bottleneck.
- Spec 076 exact packet failover baseline: 42,735 ms. Spec 077 avoids
  per-packet primary retries and uses one total deadline, but the original
  c4 node-loss campaign still had 16.67% failure and p95 14,040.745 ms.
- Fault-triggered repair run: after the A/B seed barrier, RepoA was killed;
  12/12 reads succeeded, p50 1.400 ms, p95 1,087.508 ms, and RepoC completed
  durable repair from RepoB after 43,950 ms.

These results establish correctness and bounded resource behavior, not
production-grade high throughput. Tail latency and write/control-plane
parallelism remain the next optimization target.
