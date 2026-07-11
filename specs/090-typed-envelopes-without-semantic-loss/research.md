# Research Decisions

This is contract convergence, not a new service protocol. The existing
`ProviderCapabilityHint` already contains runtime, network, lease, status, and
domain payload extension points, so introducing another envelope would add
structure without value. The simpler migration is to version it, centralize
decoding, remove duplicate producer aliases, and retain domain schemas.
