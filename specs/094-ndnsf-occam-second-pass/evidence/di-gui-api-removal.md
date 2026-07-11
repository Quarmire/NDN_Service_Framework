# DI GUI And API Simplification

Removed:

- old `RuntimeRoleProfile` / `RuntimeGuiProfile` schema and migration shim;
- duplicate Script Controller, Script User, and Script Provider tabs;
- old role-command builder and profile load/write helpers;
- duplicate role controls from the regression runner;
- unused `repo_manifests` API keywords and dual-name selector.

Retained:

- version-2 `ThreeRoleGuiProfile`;
- direct USER/PROVIDER/CONTROLLER role tabs and headless path;
- distinct policy, split, certificate, Qwen MiniNDN, and regression tools;
- one `artifact_references` API and unchanged retrieval semantics.

Verification:

```text
Tk widget tests: 9 passed
GUI/headless helper tests: 15 passed
other DI discover tests: 137 passed, 1 skipped
old symbol/API scan: zero active references
Python compilation and diff check: passed
```

The first DI discover after deletion found two accidental boundary removals,
`JsonTextPane` and `DEFAULT_POLICY`. Both are shared current components and were
restored from the pre-change commit. No old role mechanism was restored.
