# Implementation Plan: NDNSF-DI GUI Headless Automation

## Design

Keep one runtime control layer:

```text
ThreeRoleGuiProfile
    -> RoleRuntimeController
        -> RealRuntimeFactory or FakeRuntimeFactory
```

Tk widgets and CLI both use that layer. The CLI should not instantiate
`DistributedInferenceGui`, because a headless test environment may not have an
X display.

## Config Merge Rules

1. Start with `ThreeRoleGuiProfile()` defaults.
2. If `--profile` is present, load that full profile.
3. Apply role config files in Controller, Provider, User order.
4. A role config file may be either a direct role JSON object, an object wrapped
   under the role key, or a full three-role profile.

## Verification

- Python compile check.
- Non-display unit tests.
- Fake-runtime headless CLI command.
- MiniNDN launcher preflight using the headless command path.
