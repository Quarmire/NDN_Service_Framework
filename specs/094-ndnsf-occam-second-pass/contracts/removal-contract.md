# Removal Contract

For every removed mechanism:

1. Exact active caller inventory is empty or all callers are migrated in the
   same phase.
2. One named canonical replacement exists and has focused regression tests.
3. Security, persistence, wire format, and fail-closed behavior are unchanged.
4. Unsupported old input fails visibly; it does not silently select a fallback.
5. The phase is independently revertible.

Deferred compatibility requires an owner, a deadline/gate, counters where
applicable, and a future deletion task. A broad claim that compatibility may be
useful is not sufficient.
