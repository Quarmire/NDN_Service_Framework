# Implementation Plan: Targeted User Lifecycle Attribution

Add a both-end trace flag, parse existing ServiceUser lifecycle events, join by
request ID, and run one frozen cell. Python-only; core source read-only.

## Constitution Check
PASS: runtime/security unchanged, CodeGraph first, Spec Kit/GSD/ARS and MiniNDN active. Rollback is script revert.
