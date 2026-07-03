# Quickstart: Token Certificate Bootstrap

## Build

```bash
./waf configure --with-examples --with-tests
./waf build
```

## Manual Flow Regression

```bash
examples/run_hello_auth_regression.sh
```

Expected:

```text
HELLO_AUTH_REGRESSION=PASS
```

## Automatic Token Bootstrap Regression

```bash
examples/run_token_certificate_bootstrap_regression.sh
```

Expected:

```text
TOKEN_CERTIFICATE_BOOTSTRAP_REGRESSION=PASS
```

The logs should show:

```text
NDNSF_CERT_BOOTSTRAP_ISSUED identity=/example/hello/user
NDNSF_CERT_BOOTSTRAP_INSTALLED identity=/example/hello/user
NDNSF_CERT_BOOTSTRAP_REUSED identity=/example/hello/user
NDNSF_CERT_BOOTSTRAP_ISSUED identity=/example/hello/provider
NDNSF_CERT_BOOTSTRAP_INSTALLED identity=/example/hello/provider
Received response: HELLO
```

The repeat user startup should reuse the existing controller-signed certificate.
The controller log should contain only one user issuance for the same identity.
The script starts a temporary host NFD when one is not already running and only
stops the NFD instance it started.

## Token File Compatibility

NDNSF and the NDNCERT token challenge share the same first two columns:

```text
<identity> <token> [role]
```

NDNSF uses the optional role column for diagnostics. NDNCERT uses the identity
and token columns and ignores extra text.

## Python API

Native Python wrapper users can configure bootstrap directly:

```python
from ndnsf import ServiceController, ServiceProvider, ServiceUser

controller = ServiceController(
    bootstrap_token_file="examples/hello.bootstrap-tokens")
provider = ServiceProvider(
    bootstrap_name="/example/hello/provider",
    bootstrap_token="provider-token-045")
user = ServiceUser(
    bootstrap_name="/example/hello/user",
    bootstrap_token="user-token-045")
```

Process orchestration configs expose the same settings:

```python
from ndnsf import ControllerConfig, ProviderConfig, UserConfig

ControllerConfig(
    policy_file="examples/hello.policies",
    bootstrap_token_file="examples/hello.bootstrap-tokens")
ProviderConfig(
    name="provider",
    binary="App_Provider",
    bootstrap_name="/example/hello/provider",
    bootstrap_token="provider-token-045")
UserConfig(
    name="user",
    binary="App_User",
    bootstrap_name="/example/hello/user",
    bootstrap_token="user-token-045")
```

The direct Python object API can be regression-tested without using the C++
example application flags:

```bash
examples/run_python_token_certificate_bootstrap_regression.sh
```

Expected:

```text
PYTHON_TOKEN_CERTIFICATE_BOOTSTRAP_REGRESSION=PASS
```

## MiniNDN Validation

Run the HELLO MiniNDN harness with the token bootstrap flags enabled. Expected result is normal HELLO completion after user/provider certificate bootstrap.
