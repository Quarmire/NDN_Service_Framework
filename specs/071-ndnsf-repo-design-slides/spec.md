# Feature Specification: NDNSF-Repo Design Slides

**Feature Branch**: `Experimental`

**Created**: 2026-07-09

**Status**: Complete

**Input**: User description: "Create a LaTeX PDF slide deck under docs/NDNSF-REPO/slides, focused mainly on the design and mechanisms of NDNSF-DistributedRepo."

## User Scenarios & Testing

### User Story 1 - Understand the Repo Architecture (Priority: P1)

As a technical audience member, I can understand why NDNSF-DistributedRepo exists, where its responsibility ends, and how its client, service adapter, storage core, and backend fit together.

**Why this priority**: The deck must communicate the system design rather than merely list APIs.

**Independent Test**: A reader can identify the three implementation layers, the two deployment roles, and the boundary between NDNSF Core, the repo, and application-specific logic from the first half of the deck.

**Acceptance Scenarios**:

1. **Given** the compiled deck, **When** a reader reviews the architecture section, **Then** the roles of RepoClient, RepoNode, RepoCore, and RepoStoreBackend are explicit.
2. **Given** the deployment-mode slide, **When** a reader compares In-App and Persistent repos, **Then** latency, durability, and exposure-path differences are visible without implying that either mode changes remote NDNSF security semantics.

---

### User Story 2 - Follow the Data and Control Paths (Priority: P2)

As an implementer, I can follow how an application names, stores, locates, fetches, verifies, replicates, expires, deletes, and repairs one logical object.

**Why this priority**: The value of the repo comes from the interaction of its data path and object-level control plane.

**Independent Test**: The deck contains separate, readable diagrams for insertion, retrieval, placement, catalog synchronization, deletion, retention, and repair.

**Acceptance Scenarios**:

1. **Given** an application-owned segmented object, **When** a reader follows the INSERT slide, **Then** the reader sees that the repo stores opaque signed Data wire packets and does not decrypt or reinterpret them.
2. **Given** a catalog entry, **When** a reader follows the catalog slides, **Then** the reader can distinguish object-level metadata exchange from payload-segment transfer.

---

### User Story 3 - Rebuild and Present the Deck (Priority: P3)

As a project maintainer, I can rebuild a polished 16:9 PDF from the canonical LaTeX source and visually inspect every page.

**Why this priority**: The deck must remain reproducible and maintainable as the repo evolves.

**Independent Test**: Two LaTeX passes produce a PDF with the expected page count, correct page numbers, no overfull slide content, and readable diagrams in rendered page images.

**Acceptance Scenarios**:

1. **Given** the documented build command, **When** it is run in the slide directory, **Then** `main.pdf` is generated successfully.
2. **Given** rendered page images, **When** they are inspected as a contact sheet and at full size, **Then** no text overlaps, clips, or becomes unreadably small.

### Edge Cases

- A mechanism described in Python integration code but not in the C++ core must be labelled at the correct layer.
- Experimental or not-yet-scaled behavior must not be presented as a production guarantee.
- Application-specific model, UAV, video, or mission semantics must remain outside the repo design boundary.
- Large payload handling must not be confused with continuous stream publication.

## Requirements

### Functional Requirements

- **FR-001**: The deck MUST focus on NDNSF-DistributedRepo design, mechanisms, and verified integration paths.
- **FR-002**: The deck MUST distinguish NDNSF Core, NDNSF-DistributedRepo, and application responsibilities.
- **FR-003**: The deck MUST explain RepoClient, RepoNode, RepoCore, and storage backend layering.
- **FR-004**: The deck MUST explain In-App and Persistent repo deployment roles.
- **FR-005**: The deck MUST explain publisher-owned object names and the separation between service names and stored-data names.
- **FR-006**: The deck MUST explain RepoObjectManifest, RepoDataReference, INSERT/STATUS, manifest-aware fetch, and application-side verification.
- **FR-007**: The deck MUST explain capability-driven placement and failure-domain-aware replication.
- **FR-008**: The deck MUST explain object-level catalog status, snapshot, delta, lookup, and query operations.
- **FR-009**: The deck MUST explain tombstones, retention, repair eligibility, repair planning, and optional sidecar execution.
- **FR-010**: The deck MUST explain security and trust boundaries without claiming that the repo decrypts or validates opaque application payloads while storing them.
- **FR-011**: The deck MUST show how UAV and distributed-inference applications use the generic repo without moving domain policy into it.
- **FR-012**: The deck MUST summarize current MiniNDN validation markers and qualify unmeasured scale or performance claims.
- **FR-013**: The canonical source MUST be LaTeX Beamer in `docs/NDNSF-REPO/slides/main.tex` and compile to `main.pdf`.

### Key Entities

- **Slide Deck**: The ordered set of frames that explains one coherent NDNSF-DistributedRepo design story.
- **Mechanism Claim**: A statement about behavior that is traceable to current source, project documentation, or an existing regression.
- **Visual Flow**: A diagram showing one architecture, data path, state transition, or responsibility boundary.

## Success Criteria

### Measurable Outcomes

- **SC-001**: The deck contains no more than 20 frames and each content frame communicates one primary point.
- **SC-002**: All core design areas in FR-002 through FR-012 are represented by at least one frame.
- **SC-003**: Two consecutive LaTeX builds complete successfully and produce a PDF whose page count matches the frame count.
- **SC-004**: Rendered pages show no overlapping or clipped text and no unreadable diagram labels at 16:9 presentation size.
- **SC-005**: Every implementation or validation claim can be traced to current repository source, documentation, or regression commands.

## Assumptions

- Slides are in English to match the project's existing technical presentation materials.
- The visual style follows the existing white-background, dark-blue-title NDNSF decks.
- This task produces the canonical LaTeX/PDF deck only; editable PPTX and speaker notes are out of scope.
- Existing code and experiment results are documentation inputs and are not modified by this feature.
