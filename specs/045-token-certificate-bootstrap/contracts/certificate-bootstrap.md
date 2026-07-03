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

ApplicationParameters contain a TLV block:

```text
CertificateBootstrapRequest
  Identity: requested identity Name
  Token: UTF-8 string
  CertificateRequest: wire-encoded ndn::security::Certificate
```

The certificate request is the requester's locally generated certificate. It supplies the key name and public key. The controller does not receive or generate a private key.
For normal user/provider APIs, the requested identity comes from the existing
user/provider identity configuration; callers do not need to provide a separate
bootstrap name that duplicates it.

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
- Controller also checks that the Identity field inside the request equals the
  identity in the Interest name.
- Controller also checks that the supplied certificate request identity equals
  the requested identity.
- Token is consumed after successful issuance in the same controller process.
- Manual certificate flow remains supported when no bootstrap token is configured.
- User/provider startup first checks the local default certificate for the requested
  identity. If its signature KeyLocator is under the controller identity prefix,
  the requester reuses it and does not send this bootstrap Interest.
- The token file format is compatible with the NDNCERT token challenge in the first
  two columns: `<identity> <token>`. NDNSF accepts an optional third `role` column
  for diagnostics.
