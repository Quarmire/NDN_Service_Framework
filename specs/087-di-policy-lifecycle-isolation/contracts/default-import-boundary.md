# Default Import Boundary

`import ndnsf_distributed_inference as di` MUST NOT expose semantic-cache
symbols. Explicit semantic-cache imports use:

```python
from ndnsf_distributed_inference.experimental.semantic_cache import ...
```

Advisory coordination has no DI import path because the frozen retention gate
failed and its implementation was deleted.

Exact Forward Cache remains available from the normal runtime because it is a
strict provider-local exact-match optimization.
