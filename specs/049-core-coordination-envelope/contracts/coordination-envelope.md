# Contract: Coordination Envelope

## CoordinationIntent

- `intent_id`
- `request_id`
- `requester_name`
- `service_name`
- `purpose`
- `utility_weight`
- `deadline_ms`
- `nonce`
- `created_at_ms`
- `expires_at_ms`
- `payload_schema`
- `payload`
- `metadata`

## CoordinationSuggestion

- `suggestion_id`
- `intent_id`
- `request_id`
- `service_name`
- `coordinator_name`
- `window_id`
- `created_at_ms`
- `expires_at_ms`
- `proof`
- `payload_schema`
- `payload`
- `score_breakdown`

## Core Validation

Core validation covers freshness and proof. Application validation must still
check payload-specific semantics and provider authority.
