# Implementation And Regression Evidence

- The canonical Spec 095 command builder and parser gained a default-preserving
  `include_video=true` parameter.
- The new isolation campaign imports those helpers and owns only mode matrix,
  orchestration, component normalization, and cell aggregation.
- Control-only commands contain `--auto-mavlink-test` but no
  `--auto-video-test` or FEC option.
- Video-only and combined cells use the existing 60-second/900-frame gate,
  malformed-metric checks, buffering bounds, and stale checks.
- Dry-run produced exactly 15 unique runs in five cells.
- Focused tests: 7 parity campaign plus 6 isolation campaign tests passed.
- Full Python suite: 348 passed, 1 skipped.
- C++ runtime was not modified after the previously passing 215-test Spec 095
  suite, so no new C++ build was required for this experiment-only feature.
- Strict Spec Kit structure and 13/13 requirement traceability passed before
  implementation.
- No runtime, Core, UAV protocol, proposal slide, or proposal paper file was
  modified by Spec 096.
