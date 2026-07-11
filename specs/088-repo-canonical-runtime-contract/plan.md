# Implementation Plan: Repo Canonical Runtime And Contract

## Constitution Check

V2/Targeted invocation and all authorization remain unchanged. CodeGraph caller
inventory precedes moves. Exact packets, persistence, HA and MiniNDN campaigns
are mandatory acceptance evidence.

## Decision

Adopt the C++ library as the canonical object/protocol contract. Keep Python as
an adapter for NDNSF orchestration, persistent operational metadata, placement,
repair scheduling and experiments, but move that adapter into the Repo project.
Delete duplicated DI-owned Repo policy only after parity fixtures pass.

## Migration Slices

1. Freeze black-box exact packet, SQLite/cache, HA, repair and catalog fixtures.
2. Expose any missing C++ contract through `py_repoclient`.
3. Move network orchestration from DI `repo.py` into `py_repoclient` without
   changing wire names or SQLite state.
4. Replace DI imports with the public client and delete duplicate implementations.
5. Run rollback, restart, malformed, authorization and MiniNDN campaigns.

## Rollback

Each slice is a separate commit. Existing SQLite files remain readable by the
previous version; a downgrade that cannot read a schema version fails closed
without modifying the database.
