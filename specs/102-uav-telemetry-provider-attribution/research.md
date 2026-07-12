# Research
The full RequestHandler already receives requester, provider, service, and
request ID. UAV telemetry unnecessarily registered a SimpleRequestHandler.
Application-local metadata logging is sufficient; no core protocol change or
payload logging is needed. Handler absence remains ambiguous between pre-handler
security rejection and network loss; handler return proves computation only.
