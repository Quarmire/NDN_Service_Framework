# 0.5B implementation preflight

- C++ build: PASS (`./waf build -j$(nproc)`, 305 targets).
- C++ focused tests: PASS, 9/9.
- Spec 109 Python tests: PASS, 10/10.
- Packaged security contract: PASS, 1/1.
- Python binding regression after in-place rebuild: PASS, 2/2. The first
  MiniNDN attempt proved the previously built `_ndnsf.so` lacked the already
  declared `ServiceResponse.request_id` binding.
- MiniNDN fake LLM pipeline: FAIL. After the binding rebuild removed the first
  `AttributeError`, the unchanged rerun reached the request path but terminated
  after approximately 63 seconds with `local deadline`. This is preserved as a
  negative preflight result, not a GPU candidate result.
- MiniNDN/iTiger GPU candidate: NOT STARTED; exact predecessor gate is BLOCKED.

Both original failures, the post-rebuild output, and exit codes are retained under
`results/spec109-itiger-qwen/preflight/0.5B/`.
