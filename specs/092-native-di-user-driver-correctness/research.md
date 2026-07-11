# Research Notes

Spec 091 established the pre-fix evidence. Child mode completed requests but
could not sustain offered timing. Threaded mode failed scope-key retrieval while
the base publisher was not running. Process-pool completed all requests, but
its throughput denominator included a five-second worker startup lead and its
maximum schedule slip was not measured.

No external literature is needed for this correctness fix. Statistical claims
remain bounded to matched repetitions and predeclared gates.
