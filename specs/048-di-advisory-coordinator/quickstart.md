# Quickstart: Advisory Coordinator

The coordinator is optional. Existing user-side planning works unchanged:

```python
assignment = choose_runtime_assignment(template, provider_candidates, request_id="req-1")
```

To request a non-binding suggestion:

```python
coordinator = AdvisoryCoordinator(AdvisoryCoordinatorConfig(
    enabled=True,
    proof_secret="demo-secret",
))

intent = PlanIntent(
    intent_id="intent-1",
    request_id="req-1",
    user_name="/user/alice",
    template_id=template.template_id,
    nonce="n1",
)

suggestions = coordinator.suggest(
    template,
    {"intent-1": provider_candidates},
    [intent],
)

local = choose_runtime_assignment(template, provider_candidates, request_id="req-1")
assignment = merge_advisory_suggestion(
    local,
    suggestions.get("intent-1"),
    template,
    provider_candidates,
    proof_secret="demo-secret",
)
```

If the suggestion is stale, tampered, for another request/template, or points to
a provider that is no longer valid in the local ACK/lease candidate set, the
local assignment is kept.
