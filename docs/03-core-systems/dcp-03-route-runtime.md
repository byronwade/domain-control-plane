# Hosted and Self-Hosted Route Runtime

| Field | Value |
|-------|-------|
| Doc ID | `dcp-core-03` |
| Category | Core Systems |
| Status | draft |
| Version | 0.2.0-draft |
| Depends on | dcp-arch-01, dcp-core-01, dcp-arch-07 |
| Last major update | 2026-06-28 |

---

## Summary

The Route Runtime is DCP's **data plane for instant routing**. Once DNS points to a stable runtime endpoint, path, origin, weight, policy, and TLS presentation change in milliseconds for new requests — fully decoupled from DNS propagation. This is the foundation of the DNS-free hot path described in the extreme speed optimizations.

---

## Problem Statement

DNS TTLs turn every routing change into a distributed cache invalidation problem. Traditional CDNs solve this for their own customers but not as a **domain-native, provider-neutral, versioned, and rollback-capable** primitive that works for any origin.

---

## Architecture Overview

```mermaid
flowchart TB
    Client[Internet Client] --> DNS[Stable DNS (CNAME to edge)]
    DNS --> Edge[Anycast Edge POPs]
    Edge -->|RouteConfigBundle v42| Router[Router Engine]
    Router --> O1[Origin A]
    Router --> O2[Origin B]
    
    Kernel[DCP Kernel / Compiler] -->|signed immutable bundle| EdgeConfig[Edge Config Store]
    EdgeConfig -->|delta push + version pointer| Edge
    
    Edge -- gRPC / Connect stream --> Kernel
```

**Key principle**: The edge serves traffic from the **current active bundle version pointer**. Changing routing = atomic pointer swap after quorum. DNS stays stable forever after initial onboarding.

---

## RouteConfigBundle (Core Schema)

Bundles are **immutable, content-addressable, and signed**. New version = completely new bundle. Rollback = reactivate previous version pointer.

### Protobuf Definition (recommended canonical form)

```protobuf
syntax = "proto3";

package dcp.v1;

option go_package = "github.com/byronwade/dcp/proto/dcp/v1";
option java_package = "com.byronwade.dcp.v1";

message RouteConfigBundle {
  uint64 bundle_version = 1;           // Monotonic per domain/FQDN group
  string domain = 2;                     // Primary domain or wildcard pattern
  google.protobuf.Timestamp effective_at = 3;
  google.protobuf.Timestamp expires_at = 4;   // Optional soft expiry

  repeated Route routes = 5;
  TLSConfig tls = 6;
  repeated HeaderAction header_actions = 7;
  repeated PolicyRef policies = 8;       // Cedar policy references

  string signature = 9;                  // Ed25519 or similar over the bundle
  string previous_bundle_hash = 10;      // For audit/rollback chain
  string compiler_version = 11;
  string intent_hash = 12;               // Content hash of original intent
}

message Route {
  Match match = 1;
  Backend backend = 2;
  uint32 weight = 3;
  repeated HeaderAction header_actions = 4;
  HealthCheck health_check = 5;
}

message Match {
  string path_prefix = 1;
  string host = 2;                       // For virtual hosting / SNI
  map<string, string> headers = 3;       // Header-based routing
  string method = 4;
}

message Backend {
  string url = 1;                        // Resolved origin (literal IP preferred)
  string name = 2;
  bool tls = 3;
  uint32 connect_timeout_ms = 4;
  uint32 request_timeout_ms = 5;
}

message TLSConfig {
  string cert_id = 1;                    // Reference to managed cert
  string min_version = 2;                // "1.2", "1.3"
  HSTS hsts = 3;
  bool mtls_required = 4;
}

message HSTS {
  uint32 max_age = 1;
  bool include_subdomains = 2;
  bool preload = 3;
}

message HeaderAction {
  string name = 1;
  string value = 2;
  enum Action {
    SET = 0;
    APPEND = 1;
    REMOVE = 2;
  }
  Action action = 3;
}

message PolicyRef {
  string policy_id = 1;                  // Cedar policy pack ID
  string version = 2;
}

message HealthCheck {
  uint32 interval_ms = 1;
  string path = 2;
  uint32 healthy_threshold = 3;
  uint32 unhealthy_threshold = 4;
}
```

### Delta Encoding (for extreme speed)

For P0 hot path efficiency we use **delta bundles** instead of full snapshots:

```protobuf
message RouteConfigDelta {
  uint64 base_version = 1;
  uint64 new_version = 2;
  repeated RouteDelta route_deltas = 3;
  TLSConfigDelta tls_delta = 4;
  repeated HeaderDelta header_deltas = 5;
  // ... policy deltas, etc.
}

message RouteDelta {
  enum Op { ADD = 0; UPDATE = 1; REMOVE = 2; }
  Op op = 1;
  Route route = 2;                       // Full route on ADD/UPDATE
  string route_id = 3;                   // For UPDATE/REMOVE
}
```

**Why delta matters for speed**:
- Much smaller payloads at high route counts.
- Faster apply time at edge (especially Pingora/Envoy).
- Lower bandwidth to global POPs.

---

## Version Pointer & Atomic Activation

Each FQDN (or wildcard group) has a **single active version pointer**:

```json
{
  "domain": "api.example.com",
  "active_bundle_version": 42,
  "active_bundle_hash": "sha256:...",
  "last_swapped_at": "2026-06-28T...",
  "quorum_acks": ["pop-eu1", "pop-us1", "pop-ap1"]
}
```

- Pointer lives in a tiny, highly available store (etcd, NATS KV, or in-memory with Raft in Stack A).
- On commit: kernel pushes delta (or full bundle) → edge agents apply → report ACK → pointer is atomically swapped when quorum reached.
- New connections immediately see the new version. Existing connections can drain or be terminated based on policy.
- Rollback = swap pointer back to previous version (instant for new requests).

This is the mechanism that delivers <400ms p99 routing activation.

---

## Hosted Runtime (Stack A - Speed First)

| Property | Value |
|----------|-------|
| Entry | Anycast IPv4/IPv6 or `*.edge.dcp.dev` / tenant-specific |
| TLS | Terminated at edge; wildcard or per-tenant certs from DCP-managed CA |
| Protocols | HTTP/1.1, HTTP/2, HTTP/3 (QUIC) with early data |
| Implementation | Pingora (primary) or Envoy |
| Config push | gRPC/Connect persistent streams + delta encoding |

**Stable DNS pattern (set once)**:

```
api.example.com     CNAME   tenant-abc.edge.dcp.dev   TTL 3600
*.api.example.com   CNAME   tenant-abc.edge.dcp.dev   TTL 3600
```

After initial setup, **zero DNS changes** for origin changes, blue/green, canaries, or policy updates.

---

## Self-Hosted Runtime

Delivered as:
- Kubernetes operator + Envoy Gateway / Gateway API
- Lightweight single-binary edge agent (Rust/Pingora or Go/Envoy)
- WASM plugin for existing infrastructure (future)

Registration example:
```bash
dcp runtime register \
  --name prod-edge-eu \
  --endpoint https://runtime.internal:8443 \
  --mtls-cert ./agent.crt
```

Push models supported:
1. gRPC bidirectional stream (recommended for speed)
2. Pull with long polling
3. Watch on object storage (S3 signed URL + notification)

**Offline / degraded mode**: Runtime serves last known good bundle indefinitely. Rejects unsigned or out-of-sequence updates.

---

## Traffic Management Features

| Feature | DNS change required? | Effective latency | Notes |
|---------|----------------------|-------------------|-------|
| Blue/green origin switch | No | < 400ms p99 | Atomic pointer swap |
| Canary (5% → 50%) | No | < 400ms p99 | Weight change in bundle |
| Geographic / latency steering | Optional | Immediate at edge | Via multiple backends + health |
| Header-based routing | No | Immediate | Match on headers |
| Wildcard `*.staging.example.com` | Once (pre-provision) | Immediate per host | Most powerful with DNS-free hot path |
| Full apex domain onboarding | Yes (once) | After initial propagation | Then zero DNS for all changes |

---

## Health Checks & Failure Handling

Runtime performs active + passive health checks:

```yaml
health_check:
  interval_ms: 2000
  path: /health
  timeout_ms: 1500
  healthy_threshold: 2
  unhealthy_threshold: 3
```

Unhealthy backends are removed from the active pool. Kernel receives `runtime.backend_unhealthy` events via NATS/JetStream for alerting and potential auto-remediation.

**Failure modes**:
- Control plane down → Serve last bundle indefinitely (excellent availability)
- Bad signature → Reject update, keep current bundle
- All backends unhealthy → 503 + structured error + alert
- Cert issues → Block TLS; kernel triggers auto-renew workflow

---

## Observability (tied to speed goals)

Per-route / per-bundle metrics (exposed via Prometheus + OpenTelemetry):

- `dcp_runtime_requests_total{host, path, backend, bundle_version}`
- `dcp_runtime_latency_ms_histogram`
- `dcp_runtime_tls_handshake_ms`
- `dcp_runtime_active_bundle_version`
- `dcp_runtime_quorum_ack_seconds`
- `dcp_runtime_delta_apply_ms`

Tracing: `X-DCP-Transaction-Id` + `X-DCP-Bundle-Version` headers link requests back to the exact intent and provenance graph.

High-cardinality labels are managed via exemplar sampling and ClickHouse for long-term storage.

---

## Security

| Control | Implementation |
|---------|----------------|
| Origin allowlist | Only explicitly declared backends are reachable |
| mTLS to origin | Optional per-route configuration |
| WAF / policy hook | Cedar policy evaluation at edge (hot path) or gateway |
| DDoS / abuse | Platform-level (hosted) or customer-controlled (self-hosted) |
| Bundle signature verification | Ed25519 (or stronger) on every update |

---

## Related Documents

- [dcp-07-extreme-speed-optimizations.md](../02-architecture/dcp-07-extreme-speed-optimizations.md) — Delta push, version pointers, and P0 targets
- [dcp-01-transactional-domain-kernel.md](./dcp-01-transactional-domain-kernel.md) — How bundles are produced and committed
- [dcp-08-technology-stack.md](../02-architecture/dcp-08-technology-stack.md) — Pingora vs Envoy decision

---

*Version 0.2.0 — Added detailed protobuf schema, delta encoding, version pointer mechanics, and stronger ties to extreme speed goals.*