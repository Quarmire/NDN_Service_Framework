# Experiment Design Notes

- Independent variables: one-way link loss (0%, 5%) and XOR parity count (0, 1).
- Dependent variables: completion, decoded frames, gaps/timeouts, FEC recovery,
  pending buffer, RTT, and control success.
- Controls: topology endpoints, delay, bandwidth, file camera, H264 bitrate,
  width, duration, startup delay, NFD log level, and software revision.
- Repetitions: three per primary cell. Report each run and descriptive aggregate;
  do not infer statistical significance from n=3.
- Confounds: MiniNDN process startup, host CPU load, file decoder timing, and
  stochastic packet loss. Random loss is intentionally not retried away.
- Interpretation: zero-loss cells expose parity overhead; 5% cells test whether
  recovery corresponds to fewer gaps/timeouts. A failure is evidence, not a
  reason to change parameters mid-campaign.

