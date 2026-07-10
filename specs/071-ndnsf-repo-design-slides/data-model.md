# Content Model: NDNSF-Repo Design Slides

## Slide

- `title`: short mechanism-oriented title.
- `claim`: one primary sentence the audience should retain.
- `visual`: one architecture, flow, state, or comparison diagram.
- `support`: no more than three concise bullets or labels.
- `evidence`: current source, README section, or regression marker behind the claim.
- `qualification`: optional experimental or boundary note.

## Mechanism Claim

- `scope`: NDNSF Core, C++ Repo, Python/deployment control path, or application.
- `behavior`: the implemented action or invariant.
- `owner`: component responsible for the behavior.
- `not_owner`: adjacent component that must not absorb the behavior.
- `validation`: source symbol or existing regression scenario.

## Visual Flow

- `actors`: two to five named components.
- `edges`: ordered actions with directional arrows.
- `data`: object, manifest, Data reference, capability, catalog delta, or repair action.
- `boundary`: trusted local path, NDNSF remote path, storage data plane, or catalog control plane.

## Deck State

```text
outlined -> authored -> compiled -> rendered -> visually inspected -> accepted
```

An authored deck cannot be accepted until both compilation and rendered-page inspection pass.
