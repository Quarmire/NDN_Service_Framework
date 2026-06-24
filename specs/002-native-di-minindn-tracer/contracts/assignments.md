# Contract: Provider Assignments

## Default Assignment

```text
/Backbone      -> /NDNSF-DI/Tracer/provider/backbone
/Head/Shard/0  -> /NDNSF-DI/Tracer/provider/head0
/Head/Shard/1  -> /NDNSF-DI/Tracer/provider/head1
/Merge         -> /NDNSF-DI/Tracer/provider/merge
```

## Alternate Assignment

The alternate assignment may keep the same roles but use alternate provider
labels in local evidence. Full MiniNDN assignment changes must still preserve
controller permissions and role coverage.

## Validation Rules

- Every role in the plan has a provider.
- No assignment references an unknown role.
- Evidence rows use the selected assignment.
