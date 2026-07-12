# Implementation Plan: UAV Telemetry Provider Attribution

Use the existing full `ServiceProvider::RequestHandler` signature at the drone
telemetry registration seam, log metadata-only enter/return events, correlate
them in the campaign parser, and run one frozen MiniNDN cell. C++17/Python 3.8+;
Waf, unittest, Boost.Test, MiniNDN. All constitution gates PASS. Rollback is a
source revert; no wire/data migration.

## Constitution Check

PASS: canonical Targeted runtime and security are unchanged; CodeGraph was used
first; Spec Kit/GSD/ARS gates and MiniNDN validation are active.
