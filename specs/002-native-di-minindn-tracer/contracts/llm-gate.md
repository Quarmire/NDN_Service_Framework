# Contract: LLM Planner Gate

LLM planner work is blocked until native tracer evidence is accepted.

Required evidence:

- policy bundle generation passed,
- native plan loaded by C++,
- native provider/session execution passed,
- readiness and artifact negative checks passed,
- timing CSV and summary JSON were written,
- MiniNDN status was recorded, and hard MiniNDN mode either passed or recorded a clear blocker.

First allowed LLM follow-up:

- create a minimal two-stage or prefill/decode planner that emits the same native-plan contract and reuses native provider execution.
