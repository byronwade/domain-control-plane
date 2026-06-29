# Extreme Speed Optimizations

| Field | Value |
|-------|-------|
| Doc ID | `dcp-arch-07` |
| Category | Architecture |
| Status | draft |
| Version | 0.2.0-draft |
| Depends on | dcp-vision-04, dcp-core-03, dcp-arch-08, dcp-core-01 |
| Last major update | 2026-06-28 |

---

## Summary

Concrete techniques to make DCP the fastest domain system measurable — split by what physics allows vs what architecture allows. This document expands on the initial 0.1.0 foundation with deeper implementation guidance, low-level optimizations, observability for performance, benchmarking strategy, and phased rollout recommendations aligned with Stack A (Speed-First).

**Core philosophy**: Exploit the fundamental asymmetry — DNS is eventually consistent and slow to propagate globally, but *routing behind stable DNS* can be changed instantly for new requests. DCP makes routing, TLS, and policy changes feel instantaneous while keeping DNS as the stable substrate.

---

## Speed hierarchy (updated targets)

| Priority | Layer | Target (p99) | Notes |
|----------|-------|--------------|-------|
| P0 | CLI/API response (`dcp route`, `dcp ls`) | < 80ms | Local daemon + cache makes most ops zero-network |
| P0 | Routing active (new connections after commit) | < 400ms | DNS-free hot path + delta push to edge POPs |
| P0 | Rollback routing | < 400ms | Same fast path, version pointer swap |
| P1 | Authoritative DNS correct & served | < 150ms | In-memory + NOTIFY + packet cache purge |
| P1 | Global propagation p95 (default TTL=30) | < 45s | Owned resolver prefetch + probe mesh |
| P2 | TLS new host (wildcard or pre-gen) | < 3s | Pre-warmed pool + ACME on owned DNS |
| P3 | Registrar NS / glue change | hours (honest constraint) | Never in hot path; use stable anycast entrypoints |

**Philosophy**: P0 must feel "instant" to developers. P1 must be fast enough that DNS is never the bottleneck users notice.

---

## P0: DNS-free hot path (the biggest lever)

After initial onboarding, **zero DNS operations** for routing, TLS, or policy changes.

### Techniques

| Technique | Effect | Implementation notes |
|-----------|--------|----------------------|
| Stable anycast CNAME/A to edge (`edge.dcp.dev` or customer-specific) | DNS never changes on deploy | Pre-provision wildcards `*.customers.{platform}.dcp.dev` or per-tenant zones |
| Wildcard `*.{tenant}.dcp.dev` pre-provisioned at tenant creation | Tenant goes live with 0 DNS transactions | Compiler emits routing intent only; no registrar/DNS lease step |
| Compiler flag `routing_only: true` (or auto-detected) | Skip DNS lease, verify, wait, and registrar steps entirely | Fast-path classification in kernel: if `dns_ops == 0 && tls_ops == 0 && registrar_ops == 0` then bypass Temporal |
| Kernel fast path (Tokio FSM, no Temporal) | Sub-50ms compile + push | Direct gRPC/Connect to edge agents; optimistic ACK to user |
| Per-FQDN version pointer in edge config | Atomic activation across POPs | Monotonic `bundle_version`; swap pointer after quorum |

**Result**: A `dcp route` or blue-green switch can be live for new connections in <400ms p99 while DNS TTLs continue to serve the old stable pointer.

---

## P0: Edge Delta push (xDS-inspired, high performance)

Push *diffs* to the data plane instead of full configs. Use persistent connections.

### Core pattern

1. Compiler produces immutable `RouteConfigBundle` (routes + TLS + origins + policy).
2. Monotonic `bundle_version` per FQDN (or per-tenant wildcard group).
3. Delta encoding: only changed routes, certs, origin IPs.
4. gRPC persistent streams (or Connect streaming) to edge POP agents.
5. Atomic pointer swap at each POP.
6. Quorum ACK (e.g., 3/5 nearest POPs) before marking transaction `committed`.
7. Origin hostnames resolved at push time → literal IPs embedded (no per-request DNS at edge).

### Recommended frameworks (Stack A)

- **Pingora** (Cloudflare OSS, Rust): Best speed ceiling. Build custom Delta config layer on top of Pingora's excellent connection handling, `io_uring` support, and low-level control.
- **Envoy + go-control-plane** (Delta xDS native): Faster to MVP. Use for Stack B; migrate hot path to Pingora later.

**Pingora-specific optimizations**:
- Use `io_uring` where available for accept/send.
- `SO_REUSEPORT` + connection coalescing.
- Pre-built `Server` instances pooled.
- Custom Lua/WASM filters only for slow paths; keep hot path in Rust.

**Data plane micro-optimizations** (apply to both Pingora and Envoy):
- Connection pool to origins (no per-request resolver).
- TCP_FASTOPEN + TFO cookies where safe.
- kTLS (kernel TLS) to origin when supported.
- Early data / 0-RTT for QUIC/HTTP/3 repeat clients.

**Quorum & activation**:
- Lightweight gossip or simple ACK mesh between nearby POPs.
- Version pointer lives in a tiny shared-memory or etcd/Raft-protected key per FQDN.
- On rollback: simply point back to previous version (instant for new connections).

---

## P0: Grep-fast CLI & local daemon (`dcpd`)

Developers should never wait on the network for common operations.

### Design

- `dcp` binary talks to local `dcpd` daemon over **unix domain socket** (or named pipe on Windows).
- `dcpd` maintains:
  - In-memory + `redb` (or `mmap`) snapshot of recent intents, leases, and route versions.
  - Local cache of owned zones and pre-provisioned wildcards.
- Optimistic responses: return `txn_id` + expected `committed` time immediately; background confirm via NATS/JetStream.
- HTTP/3 anycast API as fallback for remote/CI use (0-RTT where safe).

### Techniques for <50ms p99 on `dcp ls` / `dcp status`

| Technique | Effect |
|-----------|--------|
| Local intent + provenance cache | Most `ls` and `status` ops are pure local reads |
| `redb` embedded KV or memory-mapped snapshot | Sub-millisecond cold starts |
| Unix socket + Tokio task per connection | < 5ms accept-to-first-byte |
| Fuzzy search via `nucleo` or `skim` | Instant grep over thousands of domains |
| Ratatui TUI watch mode | Live updates via SSE from daemon (no polling) |

**Future**: Optional embedded SQLite or DuckDB for richer local queries without network.

---

## P1: Owned authoritative DNS (fast & correct)

Never rely on third-party DNS APIs in the hot path.

### In-memory + low-latency writes

- Hickory DNS (Rust) with custom in-memory zone store for DCP-owned zones.
- Writes (zone updates from kernel) in **microseconds**.
- Raft or simple leader election per zone for HA (or multi-master with conflict-free merges for simple records).
- Default TTL = **30 seconds** (configurable per record type; never default to 3600).

### Propagation acceleration

| Technique | Effect |
|-----------|--------|
| RFC 1996 NOTIFY to resolver fleet + owned public resolver | Prefetch changed names before TTL expiry |
| `pdns_control purge` equivalent (or Hickory cache invalidate) | Instant packet cache flush on commit |
| NOTIFY-triggered prefetch in resolver | Fresh answers served in seconds, not minutes |
| Accurate TTL honoring (no gratuitous over-caching) | Predictable behavior for customers |
| Probe mesh (synthetic queries from global POPs) | Real-time visibility into propagation % and tail latency |

**Hybrid strategy**:
- Speed GA: Hickory in-memory + custom NOTIFY listener.
- MVP: CoreDNS plugin or PowerDNS HTTP API (with aggressive cache tuning).

---

## P1: Compiler & kernel shortcuts

| Technique | Effect | Notes |
|-----------|--------|-------|
| Content-addressed plan cache (hash of intent + compiler version) | Skip recompile on identical intent | Store in Dragonfly/Redis with short TTL; hit rate often >70% in practice |
| Parallel `route.push` + `dns.upsert` (when both needed) | User sees routing live first | DNS update happens in background; routing is the observable win |
| WASM AOT + Wizer snapshots + `wasm-opt -O3` | Recipe execution < 5ms overhead | Pre-init eliminates cold start; pool `Store` instances |
| Turbo verify tier (edge ACK only by default) | Skip full origin health checks on hot path | Full verify only on explicit request or policy trigger |
| Cedar policy evaluation embedded in compiler hot path | Sub-millisecond policy checks | Move heavy OPA packs to cold/enterprise path |

---

## P2: TLS speed (instant for new subdomains)

| Technique | Effect |
|-----------|--------|
| Wildcard cert per tenant zone (or per-`*.customers.dcp.dev`) | New subdomain gets TLS instantly | No per-subdomain ACME challenge |
| ACME DNS-01 on **owned** authoritative DNS | Challenge visible to CA in < 200ms | No third-party DNS propagation delay |
| Cert pre-generation pool (background workers) | Issue certs before user asks | Especially valuable for wildcard + high-churn tenants |
| TLS session tickets + QUIC early data + 0-RTT | Faster repeat handshakes | Especially powerful with HTTP/3 anycast API |
| rustls + instant-acme (Stack A) | Low-overhead issuance | Integrate with Vault/OpenBao for dynamic creds |

---

## Performance Observability (new section)

You cannot optimize what you cannot measure.

### Required signals

- **Traces**: Every `dcp route` / compile / edge push with span for compiler, kernel fast path, edge ACK, DNS (when present).
- **Metrics**: p50/p95/p99 for each priority tier; hit rate on plan cache, edge quorum success, propagation %.
- **Probe time-series** (ClickHouse): Synthetic queries from global locations recording first-success timestamp after commit.
- **Version pointer history**: Audit log of every pointer swap (who, when, previous version).

### Dashboards & alerts

- Live "time-to-routing-active" histogram.
- Propagation lag heatmap (by region).
- Cache hit rates + eviction reasons.
- Edge POP health + quorum formation time.

**Implementation**: OpenTelemetry → Tempo/Jaeger + VictoriaMetrics + ClickHouse. Custom probe agents deployed at major POPs.

---

## Benchmarking & Load Testing Strategy (new section)

### Goals
- Validate SLOs under realistic load (10k–1M domains, high churn).
- Catch regressions in fast path vs full path.
- Measure real-world propagation vs theoretical.

### Recommended tools

- **Load generation**: Custom Rust (tokio + reqwest/pingora client) or k6 for API surface; vegeta for sustained traffic.
- **Chaos**: Inject latency/failure into edge gRPC streams, NATS, Temporal.
- **Probes**: Global synthetic mesh (similar to Catchpoint or custom Cloudflare Workers + Hickory resolver).
- **Profiling**: `perf`, `eBPF` (via Pixie or native), flamegraphs on Pingora/Hickory processes.

### Phased benchmarks

| Phase | Focus | Target load |
|-------|-------|-------------|
| Prototype | Local single POP, 10k domains | Validate fast path < 400ms |
| MVP | 3 POPs, 100k domains, mixed routing + occasional DNS | End-to-end p99 < 1s for routing |
| Speed GA | 10+ global POPs, 1M+ domains, high churn | Hit all P0/P1 SLOs at scale |

Store results in ClickHouse for trend analysis and regression detection in CI.

---

## Advanced Low-Level Optimizations

- **io_uring** everywhere possible (Pingora, Hickory accept path, compiler workers).
- **Memory-mapped caches** and lock-free data structures for hot read paths (`dcpd`, edge version pointers).
- **Connection coalescing** and HTTP/3 prioritization at the edge.
- **Predictive pre-warming**: Use provenance graph + historical access patterns to pre-push likely routes to nearby POPs.
- **NUMA-aware** placement of edge agents and DNS servers.
- **Kernel bypass** options (future): DPDK or similar for extreme anycast DNS serving if needed.

These are Stack A optimizations — apply selectively in Stack B/MVP.

---

## Anti-patterns (ban these — expanded)

| Pattern | Why it destroys speed |
|---------|-----------------------|
| A record (or NS) change per deploy | TTL propagation floor (minutes to hours) |
| Default TTL 3600+ | Users experience stale routing for up to an hour |
| Full zone reload on every change | Control plane blast radius + slow |
| State-of-the-World (SotW) xDS/config push | Wastes bandwidth and CPU at scale |
| Third-party DNS/registrar API in hot path | 2–30s+ per operation, unpredictable |
| Waiting for global DNS propagation before marking `committed` | User-visible latency becomes hours instead of seconds |
| Per-request origin DNS resolution at edge | Adds latency + failure modes |
| Heavy workflow engine (Temporal) on every route-only change | Unnecessary round-trips and persistence overhead |
| No local caching in CLI/daemon | Every `ls` or status becomes network-bound |
| Synchronous ACME per new subdomain | Completely avoidable with wildcards + pre-gen |

---

## SLO table (market positioning)

| Operation | SLO (p99) | How achieved |
|-----------|-----------|--------------|
| `dcp route` / intent accepted | < 80ms | Local daemon + optimistic response |
| Routing active (new connections) | < 400ms | DNS-free + delta edge push + quorum |
| Rollback routing | < 400ms | Version pointer swap (instant for new conns) |
| Authoritative DNS correct | < 150ms | In-memory + instant purge |
| Propagation p95 (TTL=30) | < 45s | Owned resolver + NOTIFY + prefetch |
| Wildcard tenant subdomain live (routing + TLS) | < 3s | Pre-provisioned + pre-gen certs |
| `dcp ls` / cached status | < 50ms | Local `redb` + unix socket |

These numbers position DCP as dramatically faster than traditional IaC + registrar workflows.

---

## Implementation phasing for speed features

Aligns with the phased framework adoption in the technology stack doc:

| Phase | Weeks | Speed focus | Key additions |
|-------|-------|-------------|---------------|
| 0 Prototype | 1–6 | Fast path skeleton | Tokio FSM + local daemon + in-memory edge mock |
| 1 MVP | 7–16 | Reliable routing | Envoy Delta xDS + Temporal dual-path + basic cache |
| 2 Speed | 17–28 | Extreme optimization | Pingora migration, Hickory in-memory DNS, Wizer AOT, plan cache, observability mesh |
| 3 Vertical | 29–40 | Scale & polish | Predictive pre-warming, NUMA, advanced probes, full benchmark suite |

---

## Related documents

- [dcp-08-technology-stack.md](./dcp-08-technology-stack.md) — Framework choices that enable these speeds (Stack A priority)
- [dcp-04-honest-constraints.md](../01-vision/dcp-04-honest-constraints.md) — Physics limits we respect
- [dcp-03-route-runtime.md](../03-core-systems/dcp-03-route-runtime.md) — Bundle format and runtime details
- [dcp-01-transactional-domain-kernel.md](../03-core-systems/dcp-01-transactional-domain-kernel.md) — Dual-path design
- [dcp-04-blue-green-origin-switch.md](../08-examples/dcp-04-blue-green-origin-switch.md) — Concrete zero-DNS example

---

## Open questions & research

| ID | Question | Options / experiments |
|----|----------|-----------------------|
| SPEED-01 | Pingora custom Delta layer vs forking/adapting existing xDS | Build lightweight custom vs extend Pingora examples |
| SPEED-02 | Optimal quorum size & topology for global edge ACK | 3/5 nearest vs gossip vs anycast broadcast |
| SPEED-03 | Predictive pre-warming effectiveness | Measure hit rate using real provenance graphs |
| SPEED-04 | Restate vs Temporal for the *full* path latency impact | Run controlled benchmarks on saga overhead |
| SPEED-05 | Best embedded cache for `dcpd` at 100k+ domains | `redb` vs memory-mapped vs DuckDB vs custom |

---

## Next steps (recommended)

1. Expand `dcp-03-route-runtime.md` with the exact `RouteConfigBundle` protobuf + delta encoding spec.
2. Prototype the Tokio fast-path kernel + unix-socket `dcpd`.
3. Stand up a 3-POP Pingora test cluster and validate <400ms routing activation.
4. Add the performance observability signals and first probe mesh.

This document is now significantly more actionable for moving from specification to high-performance implementation.

---

*Document version 0.2.0-draft — extended with deeper techniques, observability, benchmarking, low-level optimizations, and implementation phasing.*