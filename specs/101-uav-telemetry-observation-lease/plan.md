# Implementation Plan: UAV Telemetry Observation Lease

**Branch**: `Experimental` | **Date**: 2026-07-12 | **Spec**: [spec.md](spec.md)

## Summary
Add an optional timeout override to the Ground Station Targeted helper and use
5000 ms only for telemetry status. Preserve single-in-flight ownership and all
command/security behavior; run one frozen MiniNDN cell.

## Technical Context
**Language**: C++17/Python 3.8+ | **Testing**: source contract, Python, C++, MiniNDN

**Constraints**: no command retry, safety/security change, or generic timeout change

## Constitution Check
PASS: current runtime/security preserved; CodeGraph first; Spec Kit/GSD/ARS and MiniNDN gates active.

## Structure
`GroundStationServiceContainer.inc.hpp`, campaign parser/tests, and this feature directory.

## Rollback
Source revert; no migration.
