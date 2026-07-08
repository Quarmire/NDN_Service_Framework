# UAV Operational Field Contract

This feature uses the existing `Fields` key/value representation. It does not
add new NDNSF wire names.

## MissionPlanDocument

Required keys:

- `mission_plan_schema`
- `mission_plan_id`
- `mission_plan_name`
- `mission_plan_operator`
- `mission_plan_task`
- `mission_plan_part_count`

Repeated parts use:

```text
mission_plan_part_<index>_id
mission_plan_part_<index>_role
mission_plan_part_<index>_drone
mission_plan_part_<index>_completed_by
mission_plan_part_<index>_waypoints
mission_plan_part_<index>_attempt
mission_plan_part_<index>_done
mission_plan_part_<index>_return_home
```

Optional keys:

- `mission_plan_geofence`
- `mission_plan_rally_points`
- `mission_plan_metadata`

File persistence uses the same keys in a line-oriented format:

```text
# NDNSF-UAV mission plan document
mission_plan_schema=2
mission_plan_id=<id>
...
```

Values are the existing `Fields` strings. Nested maps such as metadata continue
to use the existing encoded field format.

## UavDataProductCatalogState

Keys:

- `catalog_recording_products`
- `catalog_telemetry_log_products`
- `catalog_detection_products`
- `catalog_mission_log_products`
- `catalog_total_bytes`
- `catalog_latest_product_type`
- `catalog_latest_object_prefix`
- `catalog_latest_mission_id`
- `catalog_updated_ms`

## VehicleParameterSnapshot

Keys:

- `parameter_drone`
- `parameter_source`
- `parameter_firmware`
- `parameter_vehicle_type`
- `parameter_flight_modes`
- `parameter_count`
- `parameter_complete_percent`
- `parameter_updated_ms`
- `parameter_values` optional compact encoded field map

## OperatorAuthorityLease

Keys:

- `lease_id`
- `lease_operator`
- `lease_drone`
- `lease_scope`
- `lease_issued_ms`
- `lease_expires_ms`
- `lease_revoked`
