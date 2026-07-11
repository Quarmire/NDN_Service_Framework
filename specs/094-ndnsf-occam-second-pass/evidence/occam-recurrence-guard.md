# Occam Recurrence Guard

The old scanner reported 47 active findings, mostly false positives for
correctly owned application classes, abstract methods, optional handlers, and
typed operation status. The revised scanner:

- keeps the V1 invocation guard;
- scopes Core/application leakage by ownership;
- permits the accepted internal Repo data-plane binding;
- adds exact DI and Repo rules for mechanisms removed in Spec 094;
- ignores build-tool caches, results, agent-local files, and third-party code;
- preserves separate test/docs/history classifications.

Verification:

```text
Occam audit tests: 5 passed
active prohibited findings: 0
docs findings: 5
historical spec findings: 210
test findings: 28
```

Historical and negative-test references remain visible but do not fail the
active-source gate.
