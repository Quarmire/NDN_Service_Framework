# Regression Command Index

This file resolves every `DISCOVER` row in the umbrella regression matrix or
assigns the missing evidence to a child owner.

| Area | Exact current command/evidence | Owner/blocker |
|---|---|---|
| Core collaboration | `./build/unit-tests --run_test=DistributedInferenceAsyncRuntime,GenericDynamicApi --log_level=test_suite`; `examples/run_large_data_helper_validation.sh` | Child 085 verifies exact Boost suite names before treatment |
| Normal/Targeted MiniNDN | `python3 Experiments/NDNSF_NewAPI_Minindn_Perf.py --help`; canonical recipe remains in `README.md` and child 086 freezes exact matched flags | Child 086 |
| DI MiniNDN | `sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --out <out> --requests 2 --concurrency 1 --provider-check-timeout 60 --no-local-execution-only --full-network` | Child 085 must add coordinator-off multi-user fixture |
| Repo exact/cache | `sudo -n -E python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py --exact-packet-failover-smoke --nlsr-wait-s 10 --repo-start-wait-s 15 --output-dir <out>`; tiered command from Repo baseline | Child 088 |
| Repo HA | `sudo -n python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py --ha-campaign --campaign-duration-s 60 --campaign-rps <rps> --campaign-concurrency <n> --campaign-object-mode <mode> --campaign-replication-factor 2 --campaign-write-consistency ALL --campaign-seed <seed> --output-dir <out>` | Child 088 freezes matrix |
| UAV stream/FEC unit | `./build/unit-tests --run_test=Stream,UavProtocolState --log_level=test_suite` | Child 089 |
| UAV MiniNDN | `sudo -E timeout 160s xvfb-run -a python3 Experiments/NDNSF_UAV_GUI_Minindn.py --no-cli --no-xhost --drone-headless --camera-mode file --auto-video-test --auto-stop-seconds 6 --auto-start-delay-ms 1000 --video-bitrate-kbps 1200 --video-width 320 --output-dir <out>` | Child 089 must add structured summary |

Commands with `<out>`, `<rps>`, `<n>`, `<mode>`, or `<seed>` are templates,
not completed evidence. The child baseline must substitute concrete values.
