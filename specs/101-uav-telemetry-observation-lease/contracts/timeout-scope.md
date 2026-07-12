# Timeout Scope Contract
Only `/UAV/Telemetry/GetStatus` calls pass the 5000 ms override. All other
Targeted calls omit it and therefore use `m_timeoutMs`.
