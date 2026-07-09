# UAV QGC-Parity Field Contracts

These contracts are NDNSF-UAV-APP application payloads. They are not NDNSF core
wire types.

## VehicleParameterEditRequest

- `parameter_edit_request_id`
- `parameter_operator`
- `parameter_drone`
- `parameter_name`
- `parameter_expected_value`
- `parameter_requested_value`
- `parameter_value_type`
- `parameter_target_system`
- `parameter_target_component`
- `parameter_dry_run`
- `parameter_requested_ms`

The parameter name is limited to 16 characters to match MAVLink parameter-id
constraints.

## VehicleParameterEditResult

- `parameter_edit_request_id`
- `parameter_drone`
- `parameter_name`
- `parameter_value_type`
- `parameter_accepted`
- `parameter_applied`
- `parameter_verified`
- `parameter_reason`
- `parameter_previous_value`
- `parameter_requested_value`
- `parameter_verified_value`
- `parameter_updated_ms`

## PreflightCheckItem

- `preflight_check_id`
- `preflight_drone`
- `preflight_label`
- `preflight_category`
- `preflight_status`
- `preflight_reason`
- `preflight_blocking`
- `preflight_order`
- `preflight_updated_ms`

## MavlinkMessageSummary

Standalone keys use the `mavlink_*` prefix. Inside `UavAnalyzeSnapshot`, each
message is flattened as `message.<index>.mavlink_*`.

- `mavlink_message_name`
- `mavlink_message_id`
- `mavlink_system_id`
- `mavlink_component_id`
- `mavlink_message_count`
- `mavlink_rate_hz`
- `mavlink_last_seen_ms`

## UavAnalyzeSnapshot

- `analyze_drone`
- `analyze_link_state`
- `analyze_flight_mode`
- `analyze_mission_phase`
- `analyze_video_state`
- `analyze_parameter_cache`
- `analyze_updated_ms`
- `analyze_message_count`
- `message.<index>.*`

