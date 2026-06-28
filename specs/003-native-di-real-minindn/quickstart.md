# Quickstart: Real MiniNDN Native DI Tracer

## Build

```bash
cd /home/tianxing/NDN/ndn-service-framework
./waf configure --with-examples --with-tests
./waf build --targets=di-native-provider,di-native-plan-schema-smoke,di-native-plan-manifest-smoke,di-native-provider-session-smoke,unit-tests
```

## Quick Smoke

```bash
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke
```

## Default Assignment Evidence

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --out results/native_di_real_minindn/default \
  --assignment default
```

## Alternate Assignment Evidence

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --out results/native_di_real_minindn/alternate \
  --assignment alternate
```

## Expected Evidence

- `SUCCESS`
- `policy-bundle/native-execution-plan.json`
- `policy-bundle/service-manifest.json`
- `assignment.csv`
- `summary.json`
- `summary.txt`
- `logs/`

## Accepted Evidence From 2026-06-24

```text
/tmp/ndnsf-di-real-minindn-default     assignment=default    status=SUCCESS
/tmp/ndnsf-di-real-minindn-alternate   assignment=alternate  status=SUCCESS
/tmp/ndnsf-di-real-minindn-nonroot     non-root normal run   status=FAILURE; MiniNDN root blocker recorded
```

Provider checks use `di-native-provider --check-only --wiring-check-only` on
the assigned MiniNDN node. This validates topology placement, role assignment,
native plan/manifest loading, and provider wiring. Full native request
execution remains gated until a native tracer user driver and runnable ONNX
artifacts are available.

## Regression Checks

```bash
build/unit-tests --run_test=NativeArtifactMaterializerRejectsHashMismatch,NativeProviderReadinessAckControlsSelectionEligibility,NativeProviderHandlerExtractsOnlyFinalRoleResponse,NativeExecutionPlanGeneratedJsonDrivesProviderSessionSkeleton
build/unit-tests
```
