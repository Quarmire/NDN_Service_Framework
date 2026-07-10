# Always-On Data Plane Contract

- One producer runtime per Repo process owns a bounded number of Faces/threads.
- `activatePrefix(prefix)` registers a local Interest filter on the existing Face.
- One stable forwarding route is registered for the Repo identity.
- The lookup callback receives Interest name and `CanBePrefix` and returns an exact stored Data wire or no result.
- Returned wire bytes are parsed and their embedded Data name must satisfy the Interest.
- Exact packet wires are never re-signed.
- Opaque objects are converted once to persistent serving packets and reuse this path.
- Restart restores active prefixes before the node reports ready.
