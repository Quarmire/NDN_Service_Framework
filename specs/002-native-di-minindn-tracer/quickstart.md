# Quickstart: Native DI MiniNDN Tracer

## Build

```bash
cd /home/tianxing/NDN/ndn-service-framework
./waf configure --with-examples --with-tests
./waf build --targets=di-native-plan-schema-smoke,di-native-plan-manifest-smoke,di-native-provider-session-smoke,unit-tests
```

## Evidence Run

```bash
examples/python/NDNSF-DistributedInference/native_di_tracer/run_minindn_tracer.sh \
  --out results/native_di_tracer/latest \
  --assignment default
```

To verify the alternate provider assignment:

```bash
examples/python/NDNSF-DistributedInference/native_di_tracer/run_minindn_tracer.sh \
  --out results/native_di_tracer/alternate \
  --assignment alternate
```

## Hard MiniNDN Gate

```bash
sudo -n examples/python/NDNSF-DistributedInference/native_di_tracer/run_minindn_tracer.sh \
  --out results/native_di_tracer/minindn_gate \
  --require-minindn
```

## Expected Evidence

- `SUCCESS`
- `policy-bundle/native-execution-plan.json`
- `timing.csv`
- `summary.json`
- `summary.txt`
- `logs/`
- `assignment.csv`

## Negative/Regression Checks

```bash
build/unit-tests --run_test=NativeArtifactMaterializerRejectsHashMismatch,NativeProviderReadinessAckControlsSelectionEligibility,NativeProviderHandlerExtractsOnlyFinalRoleResponse,NativeExecutionPlanGeneratedJsonDrivesProviderSessionSkeleton
build/unit-tests
```

## Accepted Evidence From 2026-06-24

```text
/tmp/ndnsf-di-native-tracer-default     assignment=default    status=SUCCESS
/tmp/ndnsf-di-native-tracer-alternate   assignment=alternate  status=SUCCESS
/tmp/ndnsf-di-native-tracer-require     --require-minindn     status=FAILURE; MiniNDN blocker recorded because this run was non-root
```
