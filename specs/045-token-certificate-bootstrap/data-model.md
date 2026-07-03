# Data Model: Token Certificate Bootstrap

## BootstrapTokenEntry

- `identity`: NDN identity name allowed to use this token.
- `token`: opaque operator-provided one-time token string.
- `role`: optional diagnostic role, such as `user` or `provider`.
- `consumed`: process-local boolean set after successful issuance.

Validation:

- Identity must be a valid non-empty NDN name.
- Token must be non-empty.
- A consumed token cannot issue another certificate in the same controller process.
- Token files are NDNCERT-compatible in the first two columns: `<identity> <token>`.
  NDNSF may parse a third `role` column for diagnostics; NDNCERT ignores it.

## CertificateBootstrapRequest

- `identity`: requested NDN identity name derived from the requester identity
  configuration.
- `token`: provided one-time token.
- `certificateRequestWire`: wire encoding of the requester's locally generated certificate.

Validation:

- Requested identity must match the identity encoded in the Interest name.
- Token must match the configured token for the requested identity.
- Certificate request must decode as an NDN certificate.
- Certificate request identity must equal the identity encoded in the Interest name.

## CertificateBootstrapResponse

- `status`: success or failure.
- `message`: diagnostic text.
- `issuedCertificateWire`: present only on success.

Validation:

- Success response contains a controller-signed certificate.
- Failure response contains no certificate.

## Local Controller-Signed Certificate

- `identity`: requester identity.
- `certificate`: local default certificate for the identity.
- `signerKeyLocator`: KeyLocator name from the certificate signature.

Validation:

- The certificate is reusable when `signerKeyLocator` is under the ServiceController
  identity prefix.
- Reusing the certificate skips token consumption and network bootstrap.

## NDNCERT Token Challenge

- `requestedIdentity`: identity from `request.cert.getIdentity()`.
- `providedToken`: token supplied by the requester.
- `tokenMap`: configured mapping from identity to token.

Validation:

- Token must match the requested identity exactly.
