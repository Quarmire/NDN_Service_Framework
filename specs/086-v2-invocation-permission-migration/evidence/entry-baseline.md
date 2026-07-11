# Entry Baseline And ABI Decision

## ABI decision

The Experimental branch removes V1 without compatibility aliases. Existing
external callers must migrate to unified V2 `RequestService` or
`RequestServiceTargeted`. The child implementation is independently revertible.

## Baseline results

| Command | Result |
|---|---|
| `./waf build --targets=unit-tests -j$(nproc)` | PASS |
| `./build/unit-tests --log_level=message` | PASS, 210 cases, no errors |
| `python3 -m unittest discover -s tests/python -p 'test_ndnsf_core*.py' -q` | PASS, 29 tests |
| `examples/run_security_regressions.sh` | PASS |

The security aggregate passed HELLO authorization, ACK payload, custom
selection, NAC-ABE routing, one-time token negative cases, and certificate
bootstrap. This is the deletion-before baseline.
