# Deck Content Contract

The canonical deck SHOULD contain the following ordered frames. A frame may merge two adjacent items only if the one-point-per-slide rule remains true and the total stays at or below 20.

| # | Frame | Required message |
|---|---|---|
| 1 | Title | NDNSF-DistributedRepo is the project focus. |
| 2 | Why a Service-Aware Repo? | NDNSF applications need durable named objects beyond one request/response packet. |
| 3 | Responsibility Boundary | NDNSF transports and protects service calls; Repo stores/catalogs opaque objects; applications keep domain policy. |
| 4 | Layered Architecture | RepoClient, RepoNode, RepoCore, and RepoStoreBackend have distinct roles. |
| 5 | Deployment Roles | In-App and Persistent repos share semantics but differ in exposure and durability purpose. |
| 6 | Two Namespace Planes | Stable service names are separate from publisher-owned stored-data names. |
| 7 | Object and Reference Model | Manifest describes a logical object; RepoDataReference points to application-published segmented Data. |
| 8 | INSERT Data Path | Repo fetches and stores opaque wire packets and exposes operation status. |
| 9 | Payload Adapter Path | Raw bytes are segmented/signed by client helpers before using repo object semantics. |
| 10 | Fetch and Verification | Manifest-aware retrieval reassembles the object; the application verifies size/hash. |
| 11 | Capability and Placement | Filter by backup acceptance and capacity, score readiness, spread failure domains, then fill remaining replicas. |
| 12 | Object-Level Catalog | Snapshot, delta, lookup, and query operate on logical objects, not per-segment directories. |
| 13 | Catalog Synchronization | Persistent repos exchange small deltas and use snapshots for recovery. |
| 14 | Deletion and Conflict | Tombstones prevent stale AVAILABLE entries from resurrecting deleted objects. |
| 15 | Retention and Repair | Expiry suppresses repair; conservative copy-replica plans can be executed by an opted-in sidecar. |
| 16 | Security and Trust | Publisher namespace and signatures remain visible; repo storage is opaque; remote calls inherit NDNSF security. |
| 17 | Application Integration | UAV products and DI artifacts use the same generic object/catalog layer while retaining domain logic. |
| 18 | Validation and Takeaways | Existing MiniNDN regressions validate mechanisms; scalability and production performance remain future evaluation. |

## Visual Contract

- Aspect ratio is 16:9.
- Background is white; titles use Memphis blue.
- Body text is normally `\footnotesize` or larger; diagram labels are normally `\scriptsize` or larger.
- Footer page numbers use `\insertframenumber/\inserttotalframenumber`.
- No slide may contain clipped text, overlapping nodes, or a nested-card layout.
- Source paths may appear in a small evidence line, but raw code listings are avoided unless they are the single point of the slide.

## Evidence Contract

- Architecture claims trace to `NDNSF-DistributedRepo/include/` and `src/`.
- Deployment, namespace, catalog, and integration claims trace to `NDNSF-DistributedRepo/README.md` plus implementation files.
- Validation claims trace to `Experiments/NDNSF_DistributedRepo_Generic_Minindn.py` and `examples/python/NDNSF-DistributedRepo/generic_object_store/README.md`.
- Experimental limitations must be stated instead of filled with estimated numbers.
