# Acceptance Test Matrix

| Concern | Required evidence |
|---|---|
| Authorization model | unit tests for user/provider kind, epoch, replacement, exact lookup, concurrency |
| Encrypted permissions | `encrypted-permission-response` tests, wrong target/kind/plaintext/forged negatives |
| Normal invocation | HELLO auth and ACK regressions |
| Selection | custom selection and multi-provider regressions |
| NAC-ABE | attribute-routing regression |
| One-time tokens | mismatch/replay/negative regressions |
| Bootstrap | certificate bootstrap regressions |
| Targeted | Targeted bootstrap/refill and fast-path regression |
| Collaboration | collaboration unit/regression suite |
| Legacy rejection | V1/Bloom request does not invoke handler |
| Build | full C++ build and unit suite |
| Python | focused Core bindings/tests |
| Network | matched MiniNDN normal and Targeted smoke |
| Structure | forbidden-symbol scan plus CodeGraph caller/registration audit |
