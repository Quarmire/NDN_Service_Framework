# Default Import Boundary

`import ndnsf_distributed_inference as di` MUST NOT expose advisory coordinator
or semantic-cache symbols. Explicit imports use:

```python
from ndnsf_distributed_inference.experimental.advisory_coordination import ...
from ndnsf_distributed_inference.experimental.semantic_cache import ...
```

Exact Forward Cache remains available from the normal runtime because it is a
strict provider-local exact-match optimization.
