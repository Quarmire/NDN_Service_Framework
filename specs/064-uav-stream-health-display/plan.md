# Implementation Plan: UAV Stream Health Display

## Summary

Add a display-only bridge from UAV adaptive-video state to the reusable core
`StreamHealth` vocabulary. The GUI keeps all existing adaptive-video details and
adds a short `stream_health=...` summary where operators already see video
statistics.

## Design

1. Add `VideoAdaptiveState::streamHealthSummary(...)` that calls the existing
   `toStreamHealth(...)` helper and formats core health state, reason, pressure,
   fetch window, lookahead, next sequence, and loss/gap counters.
2. Append the new summary beside `compactSummary()` in the video stats line.
3. Add `video_stream_health` map/inspector tags beside existing
   `video_adaptive` tags.
4. Keep all domain-specific adaptive-video details visible and unchanged.

## Verification

```bash
./waf build --target=unit-tests
./build/unit-tests --run_test=UavProtocolState
git diff --check
```
