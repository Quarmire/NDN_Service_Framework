# T096 Runtime safety analysis

Status: **PASS with bounded tooling limitations**.

## Address/undefined behavior sanitizer

A dedicated harness compiles the production `DependencyWaitScheduler.cpp` with
Clang AddressSanitizer and UndefinedBehaviorSanitizer, then executes 1,000
waits with four fixed workers and checks terminal cleanup:

```bash
clang++ -std=c++17 -O1 -g -fsanitize=address,undefined \
  -fno-omit-frame-pointer -pthread -I. \
  tests/sanitizer/dependency-wait-scheduler-sanitizer.cpp \
  NDNSF-DistributedInference/cpp/ndnsf-di/DependencyWaitScheduler.cpp \
  -o /dev/shm/spec105-dependency-wait-sanitizer
ASAN_OPTIONS=detect_leaks=1:halt_on_error=1 \
UBSAN_OPTIONS=halt_on_error=1 \
  /usr/bin/time -v /dev/shm/spec105-dependency-wait-sanitizer
```

Result: `DEPENDENCY_WAIT_SCHEDULER_ASAN_UBSAN_PASS completions=1000 workers=4`;
no ASan, leak or UBSan report; exit 0; peak RSS 7,892 KiB; zero swap.

## Focused execution-path resource analysis

Twelve exact production unit cases were run sequentially under
`/usr/bin/time -v`: four tensor codec cases, three bounded scheduler cases,
attempt authority, cancellation/supersede, two execution-evidence cases and
provider token restart/replay. All exited 0; peak aggregate child RSS 27,716
KiB; zero swap. The earlier 500-iteration race loop also completed without a
failure after commit `17380fa`.

The full 242-case suite separately passed at peak RSS 43,268 KiB with zero swap;
T077 retains the 1,000 pending-wait thread/memory/state-cleanup measurements.

## Limitations

- `valgrind` is not installed.
- Waf supports `--with-sanitizer`, but the current build tree is 3.2 GiB while
  the filesystem had only 2.7 GiB free. A second whole-tree sanitizer build was
  not attempted because it could reproduce the earlier disk-full corruption.
- Tensor codec, execution evidence, cancellation and provider restart received
  exact unit/resource coverage, but only the standalone bounded scheduler was
  instrumented with ASan/UBSan.
- ThreadSanitizer and live provider restart under MiniNDN were not run. T078
  already records that live network fault evidence is BLOCK rather than PASS.

These limitations prevent a broad sanitizer claim but do not hide an observed
sanitizer failure; none occurred in the supported focused analysis.
