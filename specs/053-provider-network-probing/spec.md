# Spec: Automatic Provider Network Probing

**Branch**: `053-provider-network-probing` | **Date**: 2026-07-06 | **Status**: Implemented

## Summary

Providers automatically probe peer providers for RTT/bandwidth/loss using
small data packets. Results are published via NDNSD as `PeerNetworkMetric`.
The coordinator builds a `ProviderNetworkMatrix` from these metrics and uses
edge transfer costs in placement scoring — automatically preferring co-located
providers with low-latency high-bandwidth links.

No manual configuration of co-location groups is required.

## Requirements

- REQ-053-001: `ServiceProvider` MUST support `startProviderProbing(serviceName, intervalSeconds)`.
- REQ-053-002: Probe MUST send 4 Interest/Data pairs per peer (1KB, 10KB, 100KB, 500KB) and compute rtt_ms, bandwidth_mbps, loss_rate.
- REQ-053-003: Probe results MUST be published via NDNSD `peerMetrics` meta field.
- REQ-053-004: Coordinator MUST read peer metrics from NDNSD and build `ProviderNetworkMatrix`.
- REQ-053-005: Placement scoring MUST include `transfer_cost_ms(src, dst, bytes)` in the per-candidate score.
- REQ-053-006: Co-located providers must be automatically detected and preferred without manual config.
