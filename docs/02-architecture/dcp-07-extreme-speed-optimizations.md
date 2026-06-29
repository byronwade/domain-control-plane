# Extreme Speed Optimizations

| Field | Value |
|-------|-------|
| Doc ID | `dcp-arch-07` |
| Category | Architecture |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-vision-04, dcp-core-03, dcp-arch-08 |

---

## Summary

Concrete techniques to make DCP the fastest domain system measurable — split by what physics allows vs what architecture allows.

---

## Speed hierarchy

| Priority | Layer | Target |
|----------|-------|--------|
| P0 | CLI/API response | p99 <100ms |
| P0 | Routing active (new connections) | p99 <500ms |
| P1 | Authoritative DNS correct | p99 <200ms |
| P1 | Global propagation p95 (TTL=30) | <60s |
| P2 | TLS new host | <5s |
| P3 | Registrar NS change | hours (honest) |

---

## P0: DNS-free hot path

After onboarding, **zero DNS ops** for routing changes.

| Technique | Effect |
|-----------|--------|
| Stable anycast CNAME/A to edge | DNS never changes on deploy |
| Wildcard `*.customers.{platform}` pre-provisioned | Tenant live with 0 DNS txns |
| Compiler `routing_only: true` | Skip DNS lease, verify, wait |
| Kernel fast path (no Temporal) | Tokio task → edge gRPC only |

---

## P0: Edge Delta push

| Technique | Effect |
|-----------|--------|
| Delta bundles (xDS-inspired) | Push diff not full config |
| gRPC persistent streams to POPs | No poll lag |
| Per-FQDN version pointer swap | Atomic activation |
| Quorum ACK (3/5 POPs) | Commit routing before global DNS |
| Origin DNS resolved at push | No per-request resolver |

**Frameworks:** Pingora (A) or Envoy Delta xDS (B) — see [dcp-arch-08](./dcp-08-technology-stack.md).

---

## P0: Grep-fast CLI

| Technique | Effect |
|-----------|--------|
| `dcpd` local daemon (unix socket) | <20ms accept |
| Optimistic API response | Return `txn_id` before edge ACK |
| Local intent cache (`redb`) | `dcp ls` often zero-network |
| HTTP/3 anycast API | 0-RTT where safe |

---

## P1: Owned authoritative DNS

| Technique | Effect |
|-----------|--------|
| In-memory zone store | Writes in microseconds |
| Raft quorum per zone | <10ms internal |
| RFC 1996 NOTIFY to resolver fleet | Prefetch before TTL expiry |
| `pdns_control purge` equivalent | Instant packet cache invalidate |
| Default TTL 30 (not 3600) | Lower propagation floor |

---

## P1: Owned resolver (`ns.dcp.dev`)

| Technique | Effect |
|-----------|--------|
| NOTIFY-triggered prefetch | Fresh answers in seconds |
| Accurate TTL honoring | No gratuitous over-cache |
| Customer optional adoption | Fast path for those who opt in |

---

## P1: Compiler/kernel shortcuts

| Technique | Effect |
|-----------|--------|
| Content-addressed plan cache | Skip recompile |
| Parallel `route.push` + `dns.upsert` | User sees routing first |
| WASM AOT + Wizer snapshots | Recipe exec <5ms overhead |
| Turbo verify tier | Edge ACK only by default |

---

## P2: TLS speed

| Technique | Effect |
|-----------|--------|
| Wildcard cert per tenant zone | New subdomain instant TLS |
| ACME DNS-01 on owned auth DNS | Challenge visible <200ms |
| Cert pre-generation pool | Issue before user asks |
| TLS session tickets + QUIC | Faster repeat handshakes |

---

## Anti-patterns (ban these)

| Pattern | Why slow |
|---------|----------|
| A record per deploy | TTL propagation |
| TTL 3600 default | 1h cache floor |
| Full zone reload | Control plane blast |
| SotW xDS | Full config push |
| Third-party DNS API in hot path | 2–30s per op |
| Wait global DNS before `committed` | User waits hours |

---

## SLO table (market positioning)

| Operation | SLO |
|-----------|-----|
| `dcp route` accepted | p99 <100ms |
| Routing active | p99 <500ms |
| Rollback routing | p99 <500ms |
| Authoritative DNS | p99 <200ms |
| Propagation p95 (TTL=30) | <60s |
| Wildcard tenant subdomain | p99 <10s |
| `dcp ls` cached | p99 <50ms |

---

## Related

- [dcp-08-technology-stack.md](./dcp-08-technology-stack.md) — framework choices
- [dcp-04-honest-constraints.md](../01-vision/dcp-04-honest-constraints.md) — physics limits
- [dcp-04-blue-green-origin-switch.md](../08-examples/dcp-04-blue-green-origin-switch.md) — zero DNS example