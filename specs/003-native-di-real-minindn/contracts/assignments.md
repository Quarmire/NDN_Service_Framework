# Contract: Real MiniNDN Native DI Assignments

## Default Assignment

```text
/Backbone      -> /NDNSF-DI/Tracer/provider/backbone      on ucla
/Head/Shard/0  -> /NDNSF-DI/Tracer/provider/head0         on arizona
/Head/Shard/1  -> /NDNSF-DI/Tracer/provider/head1         on wustl
/Merge         -> /NDNSF-DI/Tracer/provider/merge         on neu
```

## Alternate Assignment

```text
/Backbone      -> /NDNSF-DI/Tracer/alt-provider/backbone  on neu
/Head/Shard/0  -> /NDNSF-DI/Tracer/alt-provider/head0     on ucla
/Head/Shard/1  -> /NDNSF-DI/Tracer/alt-provider/head1     on arizona
/Merge         -> /NDNSF-DI/Tracer/alt-provider/merge     on wustl
```

## Validation Rules

- Every role in the generated native plan has one assignment row.
- No assignment references an unknown role.
- No assignment references a MiniNDN node missing from the topology file.
- Provider identity strings must differ between default and alternate runs.
