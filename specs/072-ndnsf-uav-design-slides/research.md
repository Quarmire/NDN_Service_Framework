# Research Notes: NDNSF-UAV Design Slides

## Implementation Evidence

- `DroneServiceContainer` registers provider-specific video, recording,
  catalog, parameter, parameter-edit, preflight, analyze, Targeted MAVLink,
  telemetry, camera-frame, and mission services.
- `GroundStationServiceContainer` owns selected-drone state, command safety
  gates, operator authority leases, mission workflows, recording playback,
  video fetch/adaptation/FEC, and the object-detection provider.
- `FlightControllerBackend` has mock and UDP/serial implementations; MAVLink
  bytes are built on the ground-station side and forwarded by the drone.
- `VideoPublisher::publishCurrentFrame` maps H264 chunks into stream metadata
  and publishes one XOR parity symbol by default.
- `requestVideoLane`, `VideoAdaptivePolicyInput`, and
  `VideoAdaptivePolicyDecision` implement consumer-side RTT/pressure adaptation.
- `attemptAndRecoverFrame` recovers exactly one missing data shard when parity
  is available.
- Drone recording uses an embedded `RepoCore`, hybrid AES-256-GCM-at-rest
  content, a manifest, and exact named chunk retrieval.
- `Experiments/NDNSF_UAV_GUI_Minindn.py` provides headless and GUI smoke paths
  for control, telemetry, mission, video, recording, parameters, preflight,
  authority, and operator-dashboard behavior.

## DeepSeek Second-Pass Review

DeepSeek was used only as an adversarial completeness checklist. The following
suggestions were rejected because they do not match current code:

- placing MAVLink abstractions or XOR FEC inside NDNSF Core;
- modeling telemetry as a stream subscription;
- claiming one consumer per stream session;
- describing mission compensation as in-flight route merging;
- claiming automatic producer feedback on every adaptive decision;
- inventing ServiceRouter, OperatorConsole, AutopilotManager, or XorFecEncoder
  classes that do not exist.

Useful retained checks were: state boundaries must be explicit, one-loss XOR
limits must be stated, exact-name recording retrieval must be distinct from
live streaming, and the final slide must identify production gaps.
