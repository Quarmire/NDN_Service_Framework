# Data Model

`ArmedVisibilityTimeline` contains Arm response, wait begin/expiry, first drone
armed, first Ground Station armed, and class. Classes: `satisfied`,
`drone-not-armed`, `ground-telemetry-not-visible`, `final-observation-missed`, `unknown`.

`FinalObservation` is a single cached telemetry evaluation after polling ends;
it has no network side effect.
