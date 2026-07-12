# Research
ServiceProvider already emits accepted, execute-done, publish-attempt,
published, and publish-failed TRACE events. Campaign defaults suppress them.
Temporary category TRACE is the smallest diagnostic and avoids permanent hot-path logging.
