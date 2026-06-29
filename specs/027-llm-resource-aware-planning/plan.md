# Feature 027: LLM Resource-Aware Planning

## Goal

NDNSF-DI should plan language-model requests using provider resource capacity:
GPU memory, host memory, and floating-point throughput. The plan must be
reusable across requests for the same model/provider pool, and MiniNDN providers
must be able to advertise preconfigured resource metadata in ACK payloads.

## Design Rules

1. Prefer a linear multi-stage pipeline for LLMs.
2. Use provider capacity to decide how many contiguous layers each stage can
   contain.
3. Prefer fewer stage boundaries because every boundary adds activation transfer
   and dependency exchange overhead.
4. Assign each stage to a provider that can fit the stage and has strong compute
   throughput.
5. Split one stage into multiple shards only when no provider can fit the
   minimum stage. Sharding is marked as high-cost because it adds compute and
   transfer overhead.
6. Treat the generated plan as reusable: it is keyed by model identity,
   resource profile identity, and planner version.

## Architecture

### Inputs

- Model spec JSON:
  - model id and revision
  - number of transformer layers
  - memory per layer
  - fixed runtime/KV/activation overhead
  - FLOPs per layer
  - activation bytes crossing each stage boundary
- Provider profile JSON:
  - provider name and node name
  - GPU memory MB
  - host memory MB
  - TFLOPS
  - optional explicit LLM stage capacity MB

### Planner Output

The planner writes JSON with:

- `plannerKind`: `llm-pipeline`
- `planId`: stable hash for reuse
- `reusable`: true
- `stages`: linear stage assignments with layer ranges, provider, memory, FLOPs,
  latency estimate, and transfer estimate
- `dependencies`: linear stage-to-stage activation dependencies
- `shards`: empty for normal pipeline plans; populated only when forced by
  capacity limits
- `resourceProfiles`: normalized provider resource inputs

### MiniNDN ACK Metadata

Native DI providers append resource fields to ACK payloads from environment
variables:

- `gpuMemoryMb`
- `ramMemoryMb`
- `flopsTflops`
- `llmStageCapacityMb`
- `llmMaxStageLayers`
- `modelFamilies`

The current experiment uses preconfigured values; later work can replace these
with measured telemetry.

## Validation

1. Planner can generate a reusable Qwen-small plan from sample model/provider
   profiles.
2. Normal sample profile produces a linear pipeline with no shards.
3. Forced-capacity sample produces shards only when a minimum single-layer stage
   cannot fit a provider.
4. Planner can derive provider profiles from NDNSF ACK candidate payloads.
5. Re-running the same model/provider pool hits the reusable plan cache.
6. Python sources compile.
7. Native provider executable builds after ACK metadata changes.

## Implementation Review

The first implementation slice correctly added an offline resource-aware
planner and appended resource fields to native provider ACK payloads. The review
found three gaps:

1. ACK metadata was not yet consumable as planner input.
2. Plan reuse was represented by `planId`, but there was no cache path that
   proves reuse in validation.
3. `modelFamilies` needed to parse comma-separated ACK strings, not just JSON
   arrays.

The completed slice closes these gaps by adding ACK-candidate profile parsing,
cache read/write behavior keyed by `planId`, and string-safe
`modelFamilies` parsing.

## Out of Scope

- Proposal slides.
- Real GPU probing.
- Full LLM runtime execution.
- Online adaptive replanning during a request.
