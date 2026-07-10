# Deck Content Contract

The canonical deck must include these claims, expressed without overreach:

1. NDNSF-UAV-APP is a service-oriented research workload, not a certified
   autopilot ground station.
2. NDNSF Core owns generic invocation/security/stream envelopes; UAV-APP owns
   MAVLink, mission, camera, video policy, FEC, and operator semantics.
3. The Ground Station selects and invokes named services; each Drone process
   hosts independently permissioned services around local hardware adapters.
4. Known-drone MAVLink commands use Targeted invocation after an authenticated
   bootstrap and continue to use one-time tokens and replay protection.
5. Safety gates combine typed readiness/link state with operator authority.
6. Multi-drone missions use deterministic parts and application-level
   compensation for missing responses.
7. Live video is an ongoing signed Data sequence controlled by an NDNSF
   service; recordings are encrypted named objects in Repo.
8. Receiver adaptation responds to RTT, loss/timeout, future-probe, duplicate,
   and decoder-backlog pressure, while bitrate changes remain explicit.
9. Current XOR parity recovers one missing data shard per frame.
10. MiniNDN/SITL tests validate mechanisms, but production flight safety and
    long-duration hardware breadth remain future work.
