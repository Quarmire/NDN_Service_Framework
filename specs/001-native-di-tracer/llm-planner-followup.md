# LLM Planner Follow-Up Gate

The native DI tracer is the prerequisite for real LLM planner work.

## Gate

Do not expand the LLM planner beyond stub/planning metadata until the tracer path
has accepted evidence for:

- policy bundle generation,
- C++ native plan loading,
- provider readiness and artifact materialization,
- assigned-role execution through `NativeProviderHandler`/`NativeProviderSession`,
- role timing evidence with `prefetchMs`, `executeMs`, `publishMs`, and `endToEndMs`,
- MiniNDN status recorded in the tracer evidence summary.

## Next LLM Planner Tasks

After the tracer gate is accepted, the next stage should:

- choose one minimal LLM shape first, such as prefill/decode or two-stage pipeline,
- reuse the tracer policy bundle structure instead of inventing a separate path,
- keep model/data retrieval and dependency execution in native provider code,
- add LLM-specific planner scoring only after the generated plan executes,
- preserve MiniNDN as the final validation surface.

## Current Boundary

The existing LLM planner entries remain second-stage work. They may emit abstract
roles and metadata, but they are not the acceptance target for the current native
DI tracer feature.
