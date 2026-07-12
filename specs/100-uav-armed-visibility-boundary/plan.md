# Implementation Plan: UAV Armed Visibility Boundary

**Branch**: `Experimental` | **Date**: 2026-07-12 | **Spec**: [spec.md](spec.md)

## Summary

Parse drone armed timestamps alongside Ground Station state, extract the shared
cached-state predicate, and evaluate it once after the polling loop before
expiry. Run one frozen 5% MiniNDN cell without policy tuning.

## Technical Context

**Language**: C++17, Python 3.8+ | **Dependencies**: existing NDNSF/MiniNDN

**Testing**: Boost.Test, Python unittest, MiniNDN | **Storage**: tracked evidence/local results

**Platform**: Linux MiniNDN/PX4 SITL | **Scope**: Ground Station automation and campaign parser

**Constraints**: no retry, timeout/poll tuning, security/safety changes, or payload logging

## Constitution Check

All five constitution gates PASS: current Targeted runtime/security preserved,
CodeGraph used first, Spec Kit artifacts active, and MiniNDN validation required.

## Design

Extract `autoControlPrerequisiteSatisfied`, use it in normal polling and one
final cached-only read. Parse `drone.log` timestamps and report visibility class.

## Project Structure

```text
NDNSF-UAV-APP/ground-station/GroundStationWindow.inc.hpp
Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py
tests/python/test_ndnsf_uav_stream_control_isolation_campaign.py
specs/100-uav-armed-visibility-boundary/
```

## Rollback

Source revert only; no wire or data migration.
