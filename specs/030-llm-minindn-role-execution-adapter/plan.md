# Feature 030: LLM MiniNDN Role Execution Adapter

## Goal

Turn the planner-derived LLM proportional plan from feature 029 into a native
NDNSF-DI policy bundle that the existing C++ native provider/runtime tools can
execute. Keep the model small and deterministic for this step: the purpose is
to validate LLM roles, provider assignment, dependency exchange, and NDNSF
collaboration wiring, not to scale model size.

## Design

### Bundle Generation

Add a generator that reads the 029 LLM plan and writes a normal NDNSF-DI policy
bundle through `write_policy_bundle()`. The generated service keeps the current
MiniNDN service name `/Inference/NativeTracer` so the existing user driver can
be reused, but replaces the role graph with a linear LLM pipeline:

```text
/LLM/Stage/0 -> /LLM/Stage/1 -> /LLM/Stage/2
```

Each stage is assigned to the provider selected by the proportional planner:

```text
2GB provider -> 4 layers
4GB provider -> 8 layers
8GB provider -> 16 layers
```

The bundle uses the existing small Qwen-derived ONNX files as materialization
placeholders and runs providers with the deterministic runner. This keeps the
execution path real while avoiding a larger model change.

### Runtime Adapter

The previous MiniNDN harness and local C++ smoke path assumed four fixed
NativeTracer roles. Feature 030 removes that assumption from the common paths:

- assignment CSV generation follows the roles in `native-execution-plan.json`;
- local manifest smoke can read provider assignments from CSV;
- provider check/serve commands can opt into the deterministic runner;
- full-network role timing validation compares against plan roles, not a fixed
  `/Backbone`, `/Head/Shard/*`, `/Merge` set.

## Validation

- Generate an LLM proportional policy bundle.
- Validate the generated native plan schema expects `llm`, `onnx`, and
  `llm-pipeline`.
- Run local C++ manifest execution using the assignment CSV and verify every
  `/LLM/Stage/*` role completes.
- Run provider check for every generated provider row with deterministic
  runner enabled.
- Run MiniNDN full-network mode with controller, providers, and user driver.
- Compile changed Python files and rebuild affected C++ examples.

## Residual Risk

This feature still uses deterministic execution for the small LLM stages. It
now proves the NDNSF native role/dependency wiring in MiniNDN full-network mode,
but a later campaign should compare greedy and proportional LLM layouts under
multiple request rates and then replace deterministic stage runners with real
ONNX stage artifacts once those are available.
