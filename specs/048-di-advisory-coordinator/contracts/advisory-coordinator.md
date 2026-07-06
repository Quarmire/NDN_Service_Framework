# Contract: Advisory Coordinator

## PlanIntent

Fields:

- `intent_id`
- `request_id`
- `user_name`
- `template_id`
- `utility_weight`
- `deadline_ms`
- `nonce`
- `created_at_ms`
- `expires_at_ms`
- `metadata`

## AdvisorySuggestion

Fields:

- `suggestion_id`
- `intent_id`
- `request_id`
- `template_id`
- `role_assignments`
- `coordinator_name`
- `window_id`
- `created_at_ms`
- `expires_at_ms`
- `proof`
- `score_breakdown`

## Acceptance Rules

A user may accept a suggestion only when:

- The suggestion is fresh.
- The proof matches when proof verification is configured.
- The request and template match the local assignment.
- Every role has a suggested provider.
- Every suggested provider is valid under the user's current ACK metadata and
  lease offers.

Provider execution still depends on normal NDNSF authorization and provider
lease validation.
