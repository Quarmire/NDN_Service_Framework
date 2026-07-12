# Baseline Evidence

- Run 02: Arm response `1783833705306`; drone armed `1783833705655`;
  Ground Station armed wait expired `1783833715445` with no armed telemetry.
  Classification: `ground-telemetry-not-visible`.
- Run 05: Arm response `1783833765926`; drone armed `1783833765981`;
  Ground Station armed telemetry `1783833775887`; wait expired
  `1783833775958`, about 71 ms later. Classification: `final-observation-missed`.

Pre-audit hypothesis: one final cached read fixes only the second local boundary;
it cannot repair missing telemetry delivery and must not be presented as such.
