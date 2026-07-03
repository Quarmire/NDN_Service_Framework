# NDNSF-DI Runtime Workflow

Use this document as the normal entry point for NDNSF-DI experiments. The
canonical source of runtime configuration is the runtime profile; avoid calling
individual experiment scripts directly unless you are debugging one script.

## Canonical Profile

The default DI profile is:

```bash
examples/di-native-tracer.runtime.json
```

It records the NativeTracer harness, topology, Qwen tiny proportional model
artifacts, 2GB/4GB/8GB provider profiles, runtime knobs, token settings,
requests, concurrency, target RPS, and timeouts.

## Normal Flow

1. Validate the profile before a long run:

```bash
python3 tools/ndnsf_runtime.py di validate
```

2. Print the resolved profile when you need to inspect absolute paths and
   defaults:

```bash
python3 tools/ndnsf_runtime.py di print
```

3. Run the DI doctor and save the resolved configuration:

```bash
python3 tools/ndnsf_runtime.py di doctor \
  --event-log /tmp/ndnsf-di-runtime-events.jsonl \
  --write-resolved /tmp/ndnsf-di-runtime-resolved.json
```

4. Dry-run the experiment command before spending MiniNDN time:

```bash
python3 tools/ndnsf_runtime.py di run --dry-run -- --out /tmp/ndnsf-di-run
```

5. Run the experiment from the saved resolved profile:

```bash
python3 tools/ndnsf_runtime.py di run \
  --resolved /tmp/ndnsf-di-runtime-resolved.json \
  -- --out /tmp/ndnsf-di-run
```

Arguments before `--` belong to the wrapper. Arguments after `--` are passed to
the underlying experiment script and override profile defaults.

## Common Commands

Single NativeTracer harness run:

```bash
python3 tools/ndnsf_runtime.py di run -- --out /tmp/ndnsf-di-run
```

LLM full-network campaign:

```bash
python3 tools/ndnsf_runtime.py di campaign -- --runs 1 --workloads c1:1:1
```

NativeTracer rate sweep:

```bash
python3 tools/ndnsf_runtime.py di sweep -- --target-rps-list 0,1,2
```

Planner-only LLM proportional RPS search:

```bash
python3 tools/ndnsf_runtime.py di search -- --target-rps-list 1,5,10
```

Use `--dry-run` with `di run`, `di campaign`, `di sweep`, or `di search` when
you only want to inspect the generated command.

## What Each Step Catches

- `di validate`: misspelled keys, wrong scalar types, unsupported enum values,
  and missing DI sections.
- `di print`: the effective profile after default resolution.
- `di doctor`: missing artifacts, missing topology, missing binaries, NFD socket
  status, and the ready-to-run MiniNDN command.
- `di run`: one NativeTracer execution path.
- `di campaign`: repeated full-network LLM workload runs.
- `di sweep`: NativeTracer request-rate sweep.
- `di search`: planner-side greedy versus proportional RPS search.

## When To Use Lower-Level Scripts

Use the wrapper first. Drop down to lower-level scripts only when you need to
debug script-specific behavior. The lower-level scripts still accept:

```bash
--runtime-profile examples/di-native-tracer.runtime.json
--runtime-resolved /tmp/ndnsf-di-runtime-resolved.json
```

Command-line flags on those scripts override profile defaults.

## Result Hygiene

For each meaningful run, keep the result directory and the summary files it
produces, such as JSON, CSV, lifecycle traces, or campaign summaries. If a run
is only a failed smoke or local troubleshooting attempt, delete it after the
useful finding is documented.
