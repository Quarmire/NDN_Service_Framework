# Feature Specification: Token Certificate Bootstrap

**Feature Branch**: `045-token-certificate-bootstrap`

**Created**: 2026-07-02

**Status**: Implemented

**Input**: ServiceController should keep the current manual certificate flow and also support automatic certificate signing with a configured name/token pair. User and provider APIs should use their existing identity/prefix configuration plus a token, obtain a controller-signed certificate, and then continue with the existing permission and service flow. NDNSF should absorb the NDNCERT token-challenge idea directly: a token authorizes exactly one requested identity name, while the ServiceController performs the certificate signing in the NDNSF control plane.

## User Scenarios & Testing

### User Story 1 - Token-Based Controller Signing (Priority: P1)

A provider or user starts with a local key and a configured token. Before fetching permissions, it asks the ServiceController to sign its identity certificate. If the token matches the requested name, the controller returns a controller-signed certificate and the requester installs it locally.

**Why this priority**: It removes manual certificate preparation while preserving private-key locality.

**Independent Test**: Start ServiceController with a token file, start provider/user with matching tokens, and verify both install controller-signed certificates before permission fetch.

**Acceptance Scenarios**:

1. **Given** a configured token for `/example/hello/user`, **When** App_User starts with that token, **Then** it receives and installs a controller-signed certificate for `/example/hello/user`.
2. **Given** a configured token for `/example/hello/provider`, **When** App_Provider starts with that token, **Then** it receives and installs a controller-signed certificate for `/example/hello/provider`.
3. **Given** the requester already has a local certificate for its identity signed by the controller identity, **When** it starts again with the same token, **Then** it reuses the local certificate and does not send another certificate bootstrap request.

---

### User Story 2 - Manual Flow Compatibility (Priority: P2)

Existing deployments that already prepare certificates manually should continue to run without any bootstrap token configuration.

**Why this priority**: Proposal and experiment scripts already depend on the current manual/self-created local identity path.

**Independent Test**: Run the existing HELLO auth regression without token flags and confirm behavior remains unchanged.

**Acceptance Scenarios**:

1. **Given** no bootstrap token file and no token options, **When** controller, provider, and user start, **Then** they use the existing identities and permission flow.
2. **Given** an invalid token, **When** the requester asks for a certificate, **Then** the controller refuses to issue and logs the reason without crashing.

---

### User Story 3 - NDNCERT Token Challenge Compatibility (Priority: P3)

NDNCERT-style token validation should bind a configured token to the requested
identity name. NDNSF uses that same model inside the ServiceController signer,
and the standalone NDNCERT challenge module uses the same token file format for
compatibility.

**Why this priority**: It keeps NDNSF's simplified controller CA flow aligned with the NDN certificate management tool.

**Independent Test**: Build ndncert with the token challenge module and unit-test successful, wrong-token, and wrong-name challenge outcomes.

**Acceptance Scenarios**:

1. **Given** an NDNCERT token map containing name/token pairs, **When** the requester provides the correct token for the requested name, **Then** the challenge succeeds.
2. **Given** the wrong token or a token for another name, **When** the challenge runs, **Then** the request remains failed or challenged.

### Edge Cases

- Token exists but is used for a different identity name.
- Token is reused after successful issuance in the same controller process.
- Requester certificate wire cannot be decoded.
- Requester has no network route to the ServiceController certificate endpoint.
- Controller has no token file, in which case automatic bootstrap is disabled and manual flow still works.

## Requirements

### Functional Requirements

- **FR-001**: ServiceController MUST expose a certificate bootstrap Interest prefix under its controller prefix.
- **FR-002**: ServiceController MUST load an optional identity-to-token map from configuration.
- **FR-003**: ServiceController MUST issue a certificate only when the request name, request payload identity, token-table identity, and supplied certificate request identity all match.
- **FR-004**: ServiceController MUST sign issued certificates using the controller identity and MUST NOT handle requester private keys.
- **FR-005**: User and provider startup MUST support an optional token-based automatic certificate bootstrap before permission fetch, deriving the requested bootstrap name from the configured user/provider identity instead of requiring a duplicate API argument.
- **FR-006**: Existing startup without token options MUST continue to work.
- **FR-007**: The bootstrap flow MUST log success, refusal reason, and installed certificate names.
- **FR-008**: NDNCERT MUST include a token challenge module that can validate a token bound to a requested identity.
- **FR-009**: A regression script MUST verify the full controller/provider/user automatic bootstrap path.
- **FR-010**: MiniNDN validation MUST run the automatic bootstrap path before normal service invocation.
- **FR-011**: User and provider token bootstrap MUST first check the local KeyChain for an existing controller-signed certificate and MUST reuse it instead of consuming another token.
- **FR-012**: The NDNSF controller token file and NDNCERT token challenge file MUST share the same first two columns, `<identity> <token>`, with any third role column treated as optional metadata by NDNSF and ignored by NDNCERT.
- **FR-013**: Python `ServiceController`, `ServiceProvider`, and `ServiceUser` APIs MUST expose certificate bootstrap configuration without requiring callers to hand-write example command-line flags or repeat the same identity name twice.
- **FR-014**: Python process orchestration configs MUST expose the same controller token file and user/provider token fields while preserving existing `args` and `extra_args` escape hatches.
- **FR-015**: A direct Python object-API smoke test MUST verify controller/provider/user token bootstrap and certificate reuse without invoking the C++ example applications.
- **FR-016**: Automatic certificate bootstrap requests MUST encrypt the name-bound token and certificate request to the Controller certificate before sending them in Interest ApplicationParameters.
- **FR-017**: Automatic certificate bootstrap requests MUST include a requester proof signature over the requested identity, token, certificate request, and nonce; ServiceController MUST verify that proof against the included certificate request before issuing a certificate.

### Key Entities

- **Bootstrap Token Entry**: Requested identity name, token string, optional role, and consumed state.
- **Certificate Bootstrap Request**: Requested identity, token, and requester certificate wire encoding.
- **Encrypted Certificate Bootstrap Request**: RSA-wrapped AES-CBC envelope carrying a Certificate Bootstrap Request for the Controller.
- **Issued Certificate**: Controller-signed certificate copied from the requester public key.
- **NDNCERT-Style Token Challenge**: A name-bound token validation model that accepts a token only for the certificate identity it is configured for.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Existing HELLO auth regression passes without token flags.
- **SC-002**: New token certificate bootstrap regression passes for both user and provider.
- **SC-003**: A wrong-token regression path refuses issuance and records a diagnostic reason.
- **SC-004**: The MiniNDN HELLO path completes with automatic certificate bootstrap enabled.
- **SC-005**: NDNCERT token challenge unit coverage passes or, if full ndncert tests are unavailable, the module builds and a targeted smoke test validates token matching.
- **SC-006**: A repeat user/provider startup with the same token logs certificate reuse and does not produce a second controller issuance for the same identity.
- **SC-007**: Python API/config tests verify that bootstrap token settings are passed to the native binding or generated process command.
- **SC-008**: A direct Python controller/provider/user smoke test completes HELLO once after token issuance and once after local certificate reuse.
- **SC-009**: Controller logs show encrypted bootstrap requests and valid requester proof for successful token issuance.
- **SC-010**: A bootstrap request with an encrypted payload but tampered requester proof is rejected with `request-proof-invalid`, and the same token can still be used afterward for a valid issuance.

## Assumptions

- The first implementation uses an explicit token file rather than interactive token printing.
- A certificate is considered reusable for this bootstrap path when its signature KeyLocator name is under the controller identity prefix.
- Tokens are one-time within a running ServiceController process; durable token consumption can be added later.
- The bootstrap Interest carries the requester certificate request in ApplicationParameters.
- The issued certificate is installed into the same local key in the requester KeyChain.
