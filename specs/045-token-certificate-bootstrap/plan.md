# Implementation Plan: Token Certificate Bootstrap

**Branch**: `045-token-certificate-bootstrap` | **Date**: 2026-07-02 | **Spec**: `specs/045-token-certificate-bootstrap/spec.md`

## Summary

Add a complete automatic certificate bootstrap flow to NDNSF using a built-in
ServiceController certificate signer. User/provider keep their current manual
flow, but when configured with a token they first request a controller-signed
certificate, install it locally, and then continue with permission bootstrap
and service invocation. The wire exchange remains a small NDNSF TLV endpoint,
while the authorization semantics match the NDNCERT token challenge: a token is
valid only for the exact requested identity name.

## Technical Context

**Language/Version**: C++17, Bash regression scripts, existing Python wrappers left compatible

**Primary Dependencies**: ndn-cxx KeyChain/Face/Certificate, NDNSF ServiceController/User/Provider, ndncert token challenge semantics

**Storage**: Plain text preconfigured or first-start generated identity-token map for v1; process-local issued-certificate cache for permission encryption

**Testing**: waf build, existing HELLO regressions, new token bootstrap regression, MiniNDN HELLO validation

**Target Platform**: Ubuntu 20.04/NDN local development and MiniNDN

**Project Type**: C++ library plus example applications

**Performance Goals**: Bootstrap happens once at startup and must not affect steady-state service invocation latency

**Constraints**: Do not expose private keys; do not break manual certificate flow; do not modify proposal slides; use MiniNDN for final validation

**Scale/Scope**: Configured identities and tokens for App_ServiceController/App_User/App_Provider, Python API/config surfaces, direct Python smoke, MiniNDN full-flow validation, and NDNCERT-compatible token challenge coverage

## Constitution Check

- CodeGraph used before broad code exploration.
- Spec Kit artifacts created before implementation.
- GSD health checked and workflow state kept in this plan/tasks.
- MiniNDN remains the final network validation target.

## Project Structure

### Documentation

```text
specs/045-token-certificate-bootstrap/
├── spec.md
├── plan.md
├── data-model.md
├── contracts/
│   └── certificate-bootstrap.md
├── quickstart.md
└── tasks.md
```

### Source Code

```text
ndn-service-framework/
├── CertificateBootstrap.hpp
├── CertificateBootstrap.cpp
├── ServiceController.hpp
└── ServiceController.cpp

examples/
├── App_ServiceController.cpp
├── App_User.cpp
├── App_Provider.cpp
├── hello.bootstrap-tokens
└── run_token_certificate_bootstrap_regression.sh

~/NDN/ndncert/src/challenge/
├── challenge-token.hpp
└── challenge-token.cpp
```

**Structure Decision**: Keep NDNSF automatic bootstrap as a framework helper plus
Controller endpoint because it is part of the service-control plane. Keep the
token file and identity-bound token semantics compatible with NDNCERT's token
challenge so the same operational material can be reused.

## Design Decisions

- Requesters generate or reuse a local key first, then send their self-created certificate wire to the controller. The controller copies the public key and signs a new certificate. Private keys never leave the requester.
- Requesters encrypt the CertificateBootstrapRequest to the Controller's local certificate using RSA-wrapped AES-CBC before placing it in Interest ApplicationParameters. This protects the configured token and certificate request from passive observers.
- Requesters include a proof-of-possession signature inside the encrypted request. The signature covers requested identity, token, certificate request, and nonce. The Controller verifies this proof with the certificate request public key before issuing a Controller-signed certificate.
- Token file format is line-oriented: `<identity> <token> [role]`. Lines beginning with `#` are ignored.
- Endpoint prefix is `/<controller>/NDNSF/CERTBOOTSTRAP/<identity...>`.
- ApplicationParameters carry a small TLV block with requested identity name,
  token, and certificate request wire. The requested identity is derived from
  the existing user/provider identity configuration, so application APIs do not
  require callers to repeat the same name as a separate bootstrap parameter.
- Manual flow remains the default when no token option is provided.
- The Controller treats the identity-token file as stable configuration. If a configured file path is missing, the Controller generates one from policy identities with 8-character tokens, writes it to disk, and then loads it like any other token file. Successful issuance does not mutate or consume the configured map.
- User/provider startup reuses an existing local controller-signed certificate before
  sending bootstrap Interests, so repeated starts normally avoid another signing request.
- NDNSF and NDNCERT token files share `<identity> <token>` as the compatible prefix;
  NDNSF treats a third `role` column as optional diagnostics.
- This is considered the complete NDNSF integration path for this project:
  Controller signs real NDN certificates, private keys stay local to the
  requester, token validation is bound to the requested name, and token-bearing
  bootstrap requests are encrypted to the Controller certificate.
- Python native bindings and the public Python object API expose
  `bootstrap_token_file` for controllers and only `bootstrap_token` for
  users/providers. The requested name is derived from `user` or
  `provider_prefix` plus `provider_id`. Process orchestration dataclasses emit
  only the token flag while preserving raw argument escape hatches.

## Complexity Tracking

No constitutional complexity violation. The extra helper module is justified because certificate bootstrap is shared by user/provider and should not be embedded in both example files.
