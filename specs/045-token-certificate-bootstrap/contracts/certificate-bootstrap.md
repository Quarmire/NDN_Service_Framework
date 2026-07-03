# Certificate Bootstrap Contract

## Prefix

```text
/<controller>/NDNSF/CERTBOOTSTRAP/<requested-identity...>
```

Example:

```text
/example/hello/controller/NDNSF/CERTBOOTSTRAP/example/hello/user
```

## Interest ApplicationParameters

ApplicationParameters normally contain an encrypted TLV envelope:

```text
EncryptedCertificateBootstrapRequest
  RecipientCertName: Controller certificate name
  Algorithm: RSA-WRAPPED-AES-CBC
  EncryptedAesKey: AES content key encrypted to Controller certificate public key
  Iv: AES-CBC IV
  CipherText: encrypted CertificateBootstrapRequest
```

The decrypted plaintext is:

```text
CertificateBootstrapRequest
  Identity: requested identity Name
  Token: UTF-8 string
  CertificateRequest: wire-encoded ndn::security::Certificate
  ProofNonce: random nonce
  ProofSignature: requester signature over Identity, Token, CertificateRequest, and ProofNonce
```

The certificate request is the requester's locally generated certificate. It supplies the key name and public key. The controller does not receive or generate a private key.
For normal user/provider APIs, the requested identity comes from the existing
user/provider identity configuration; callers do not need to provide a separate
bootstrap name that duplicates it.

The encrypted request also carries a proof-of-possession signature made by the
requester's local key. The Controller verifies this proof with the certificate
request public key before issuing a Controller-signed certificate.

## Response Data

Content contains:

```text
CertificateBootstrapResponse
  Status: 1 for success, 0 for failure
  Message: UTF-8 diagnostic string
  IssuedCertificate: wire-encoded controller-signed Certificate, success only
```

## Security Rules

- Controller signs only if the token matches the requested identity in the Interest name.
- Controller decrypts automatic bootstrap requests with its private key before
  reading the name-bound token.
- Controller also checks that the Identity field inside the request equals the
  identity in the Interest name.
- Controller also checks that the supplied certificate request identity equals
  the requested identity.
- Controller verifies the requester proof signature with the supplied certificate request.
- Token entries are stable controller configuration. Successful issuance does
  not mutate or consume the configured identity-token map.
- If a configured token file path is missing, ServiceController creates it on
  first startup from policy identities using 8-character tokens and then loads
  that file as the stable map.
- Manual certificate flow remains supported when no bootstrap token is configured.
- User/provider startup first checks the local default certificate for the requested
  identity. If its signature KeyLocator is under the controller identity prefix,
  the requester reuses it and does not send this bootstrap Interest.
- The token file format is compatible with the NDNCERT token challenge in the first
  two columns: `<identity> <token>`. NDNSF accepts an optional third `role` column
  for diagnostics.
