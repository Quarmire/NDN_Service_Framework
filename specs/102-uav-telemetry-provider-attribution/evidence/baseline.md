# Baseline
`ServiceProvider::RequestHandler` exposes request ID, but UAV telemetry uses
`SimpleRequestHandler`. Therefore Spec 101 timeouts cannot currently distinguish
pre-handler absence from post-handler response loss. Switching handler type at
registration is the smallest diagnostic-only change.
