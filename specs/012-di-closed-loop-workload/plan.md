# Plan: DI Closed-Loop Workload Campaign

## Approach

Start with closed-loop sequential requests because the current Python
`request_collaboration` binding is synchronous. This gives a stable workload
metric without touching C++ binding internals. It also preserves the existing
single-request path because `--requests` defaults to 1.

The user driver prints:

- `NDNSF_DI_NATIVE_TRACER_USER_REQUEST` for each request.
- `NDNSF_DI_NATIVE_TRACER_USER_EXECUTION` for backward-compatible aggregate
  parsing. For `--requests 1`, this is the same as before. For `--requests N`, it
  includes workload fields.
- `NDNSF_DI_NATIVE_TRACER_USER_WORKLOAD` as an explicit workload summary.

The MiniNDN harness keeps using the aggregate execution line but records
`requestCount`, `makespanMs`, `p50Ms`, `p95Ms`, and `throughputRps` when present.

## Steps

1. Add closed-loop workload support to `user_driver.py`.
2. Extend harness CLI and `user_driver_command()` with `--requests`.
3. Extend parsing and summary fields for workload metrics.
4. Add `--requests` to `run_layout_campaign.py` rows and summaries.
5. Validate with dry-run/py_compile, single-request smoke, and closed-loop
   full-network smoke.
6. Run a small campaign under the current 75 ms provider-capacity setting.

## Expected Interpretation

Closed-loop sequential requests measure repeatability and throughput for a
single user stream. If the difference is small, the next feature should add a
native async collaboration API or C++ user driver to issue multiple outstanding
requests and measure real queueing under provider worker limits.
