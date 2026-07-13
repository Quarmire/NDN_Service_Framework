# Live Fault Record Contract

Each once-only cell records:

- candidate/campaign/cell/command digests;
- owned target PID, process group, `/proc` start time, provider/role/boot;
- trigger and injection monotonic timestamps;
- `injectionApplied` and `networkInjection`, both true for an executed cell;
- intended and observed effect;
- before/after attempt, boot, lease, wait, thread, route, and process state;
- authenticated cancel/supersede evidence;
- one authoritative result or exact terminal reason;
- cleanup proof and exit code.

The controller rejects targets not in the current campaign registry. Kill and
restart use the normal provider binary. Data-path and timing faults use only the
separate fault provider executable. Cleanup failure stops subsequent cells.
