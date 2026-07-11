# Experiment Design Notes

## Questions

1. Does the existing Targeted Arm/Takeoff/Land sequence complete at 5% loss
   when no video stream is active?
2. Does 60-second video meet the 900-frame gate at 5% loss when no control
   commands are sent?
3. For the same parity setting, does concurrent control coincide with lower
   video or control completion than the isolated cells?

## Controls And Confounds

- Fixed topology, endpoints, link properties, software revision, policy,
  camera source, bitrate, width, startup delay, and log level.
- MiniNDN random loss, host scheduling, process startup, H264 timing, and
  command-script timing remain confounds.
- The control-only cell is sequence-matched, not duration-matched, because
  padding an idle GUI to 60 seconds adds no control traffic.
- Three repetitions support descriptive localization only. They do not support
  causal interaction or significance claims.
- Failed runs are outcomes. No automatic retry, replacement, or mid-campaign
  parameter change is permitted.

## Decision Rule

- If control-only succeeds but combined control fails, concurrency is a
  plausible boundary requiring a dedicated higher-repetition follow-up.
- If control-only also fails, the Targeted control path itself is unstable at
  this loss under the current bounded timeout.
- If video-only succeeds more often than matched combined video, concurrency is
  a plausible contributor; otherwise the stream path is already the boundary.
- Any conclusion remains descriptive unless a later powered experiment confirms
  it.
