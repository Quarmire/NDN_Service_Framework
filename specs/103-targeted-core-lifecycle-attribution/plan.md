# Implementation Plan: Targeted Core Lifecycle Attribution

Add a campaign-only flag that supplies `NDNSF_APP_NDN_LOG` with ServiceProvider
TRACE, parse existing core events, and run one frozen cell. Python 3.8+ only;
core C++ is read-only. Rollback is script revert.

## Constitution Check

PASS—runtime/security unchanged, CodeGraph first, Spec Kit/GSD/ARS and MiniNDN gates active.
