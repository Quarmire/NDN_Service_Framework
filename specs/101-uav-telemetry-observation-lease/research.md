# Research

Spec 100 run 05 dispatched one telemetry request at armed-wait start. It timed
out after 10006 ms and held the per-drone in-flight guard for the whole 10-second
wait. The Arm response already reported `armed=true`; the missing observation,
not drone execution, controlled failure. A 5-second read-only lease fits two
attempts and exceeds the retained successful telemetry maximum (~3.2 s).
