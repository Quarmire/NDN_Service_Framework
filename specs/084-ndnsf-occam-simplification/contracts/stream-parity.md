# Core Stream Parity Contract

The C++ stream implementation may become canonical only after its observable
behavior is frozen against existing Python and UAV consumers.

The parity fixture must define:

- stream/session identity and schema version;
- sequence ordering, duplicate handling, missing ranges, stale-session rules;
- maximum pending count/bytes and deterministic overflow behavior;
- deadline and gap-skip inputs, defaults, and emitted events;
- health counters and adaptive-fetch inputs/outputs;
- malformed input and unknown-version behavior;
- C++ object ownership, Python binding lifetime, thread safety, and callback
  execution guarantees;
- field/default/error conversion between TLV C++ values and Python objects.

Core provides policy inputs and deterministic state transitions. UAV supplies
video-specific deadlines, FEC/frame recovery decisions, H264 assembly, ROI,
decoder backlog policy, and display labels. Static files, models, catalog
snapshots, and planned tensor bundles are not stream objects and continue to
use exact-name segmented retrieval.

Migration is blocked until the same fixture suite passes against the current
behavior oracle and the proposed C++ binding.
