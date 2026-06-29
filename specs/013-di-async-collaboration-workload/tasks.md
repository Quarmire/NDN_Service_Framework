# Tasks: DI Async Collaboration Workload

- [x] A001 Add async collaboration binding in `pythonWrapper/src/ndnsf/_ndnsf.cpp`.
- [x] A002 Add Python wrapper method in `pythonWrapper/ndnsf/service.py`.
- [x] A003 Add `--concurrency` workload mode to `user_driver.py`.
- [x] A004 Thread concurrency through MiniNDN harness summaries.
- [x] A005 Thread concurrency through layout campaign runner.
- [x] A006 Run syntax/build checks.
- [x] A007 Run full-network async smoke.
- [x] A008 Run small async campaign and record results.
- [x] A009 Update docs with interpretation and next step.

## Validation Commands

```bash
cd /home/tianxing/NDN/ndn-service-framework
PYTHONDONTWRITEBYTECODE=1 python3 - <<'PY'
from pathlib import Path
for item in [
    'examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py',
    'Experiments/NDNSF_DI_NativeTracer_Minindn.py',
    'examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py',
    'pythonWrapper/ndnsf/service.py',
]:
    path = Path(item)
    compile(path.read_text(encoding='utf-8'), str(path), 'exec')
    print('syntax-ok', item)
PY

cd pythonWrapper && PYTHONDONTWRITEBYTECODE=1 python3 setup.py build_ext --inplace

sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment default \
  --role-execution-delay-ms 75 \
  --requests 1 \
  --concurrency 1 \
  --out /tmp/ndnsf-di-sync-sanity \
  --provider-check-timeout 60

sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment default \
  --role-execution-delay-ms 75 \
  --requests 2 \
  --concurrency 2 \
  --out /tmp/ndnsf-di-childhome-workerid-stagger-c2 \
  --provider-check-timeout 60
```

## Result

The sequential sanity run passed. The concurrent smoke did not pass: with two
outstanding child-process requester identities and parent-served scope-key
large data, one request completed in about 0.55-0.58 s while the other timed out
after 60-120 s. Core trace shows late or incomplete ACK delivery before
selection, so this is now a located NDNSF/SVS ACK-selection bounded-time
concurrency boundary rather than a planner scoring result.
