# Implementation Evidence

- Ground Station accepts `--video-fec-parity-shards=0|1` and sends
  `fec_parity_shards` in the existing video-control request.
- Drone strictly parses 0 or 1, defaults to 1, reports the accepted count, and
  uses the existing publication loop for data-only or XOR-parity groups.
- Core stream types and algorithms are unchanged.
- The canonical campaign now creates deterministic loss/parity/repetition
  cells, generated matched topologies, concurrent Targeted MAVLink commands,
  per-run acceptance, and treatment-level JSON/CSV.

Verification before the primary campaign:

- UAV/Core affected targets built successfully.
- `UavVideoFecParityRequestContract` passed.
- Five campaign parser/matrix/aggregation tests passed.
- Twelve-run dry-run produced the expected first and last cell.
- One 8-second FEC-off/MAVLink MiniNDN smoke functionally passed: parity 0 was
  accepted, 180 frames decoded, Arm/Takeoff/Land succeeded, no frame gap, and
  no pending bytes. The first parser classification exposed an end-marker
  assumption; re-parsing the same immutable logs after accepting both
  `stop-requested` and `stop-ack` classified it correctly. The run was not
  repeated.

