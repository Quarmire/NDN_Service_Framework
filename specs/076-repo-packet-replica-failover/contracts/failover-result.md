# Contract: Exact Packet Failover Result

```json
{
  "schema": "ndnsf-repo-exact-packet-failover-minindn-v1",
  "replicaNodes": ["/repo/A", "/repo/B"],
  "packetNames": ["/data/.../seg=0"],
  "expectedPacketWireSha256": ["<sha256>"],
  "actualPacketWireSha256": ["<sha256>"],
  "attempts": [
    {"repoNode": "/repo/A", "packetName": "/data/.../seg=0", "success": true},
    {"repoNode": "/repo/A", "packetName": "/data/.../seg=1", "success": false},
    {"repoNode": "/repo/B", "packetName": "/data/.../seg=0", "success": true}
  ],
  "latencyMs": 0,
  "failoverLatencyMs": 0,
  "checks": {
    "primaryOnePacketBeforeFailure": true,
    "primaryFailureObserved": true,
    "secondaryRestartedWholeSet": true,
    "exactNames": true,
    "wireIdentity": true
  },
  "passed": true
}
```
