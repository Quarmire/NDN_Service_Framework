# Deck Data Model

## SlideRecord

| Field | Meaning |
| --- | --- |
| `title` | One mechanism or design decision |
| `claim` | The one sentence the audience should retain |
| `diagram` | TikZ representation of actors, state, or data flow |
| `supporting_points` | At most four short implementation facts |
| `evidence` | Current files, symbols, specs, or harnesses |
| `boundary` | Optional limitation or core/app ownership statement |

## Mechanism Groups

- **Control plane**: bootstrap, service invocation, Targeted commands,
  telemetry, authority, and command state.
- **Mission plane**: mission document, assignment, MAVLink upload, progress,
  and compensation.
- **Continuous data plane**: H264 packets, stream/session metadata, adaptive
  fetch, reorder/skip, and XOR recovery.
- **Durable data plane**: recording manifest, encrypted repo chunks, catalog,
  and exact-name retrieval.

## Visual Invariants

- Blue denotes NDNSF/control structure.
- Green denotes healthy/accepted/durable state.
- Orange denotes adaptation, policy, or application responsibility.
- Red denotes rejection, stale state, or an explicit boundary.
