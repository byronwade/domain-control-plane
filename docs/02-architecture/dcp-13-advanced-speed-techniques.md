# Advanced Speed Techniques

| Field | Value |
|-------|-------|
| Doc ID | `dcp-arch-13` |
| Category | Architecture |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-07, dcp-arch-08, dcp-core-11, dcp-core-12 |
| Created | 2026-06-28 |

---

## Summary

This document collects **next-level speed techniques** beyond the foundational extreme optimizations. These are higher-order ideas that become relevant once the basics (delta push, version pointers, DNS-free hot path, Pingora + custom layer) are in place. They target tail latency, predictability, hardware efficiency, and intelligent routing.

These techniques help DCP compete with (and potentially surpass) the best edge platforms while staying true to our domain-native model.

---

## 1. Predictive Pre-warming & Affinity Routing

Instead of reacting to traffic, proactively push likely bundles and warm state to the POPs a user is most likely to hit.

**Techniques**:
- Use provenance graph + historical access patterns to predict next likely routes/domains.
- Pre-warm WASM modules and Domain State caches at nearby POPs.
- Affinity routing: pin users/sessions to specific POP clusters when beneficial (with fallback).
- Background "speculative" execution of common intents.

**Impact**: Dramatically reduces cold-path latency for repeat users and high-churn tenants.

---

## 2. Hardware-Aware Optimizations

### NUMA & CPU Affinity
- Pin edge agents and DNS servers to specific NUMA nodes.
- Use `taskset` / `numactl` or Rust equivalents in production deployment.
- Hugepages for large route tables and state stores.

### io_uring Deep Integration
- Use io_uring not just for accept/send, but for the entire hot path where possible (including state access).
- Custom io_uring-based connection pool and timer management.

### Memory & Zero-Copy Paths
- Zero-copy where possible between kernel fast path and edge (shared memory rings or `io_uring` registered buffers).
- Lock-free data structures for version pointers and route tables (crossbeam, etc.).

---

## 3. Tail Latency Reduction

P99 is good. P999 and P9999 win in production.

**Techniques**:
- Request hedging for non-critical operations.
- Adaptive timeouts + early rejection of slow backends.
- Per-POP load shedding based on real-time health signals.
- Priority queuing at the edge (HTTP/3 prioritization + custom scheduler).
- Continuous profiling + automated flamegraph analysis in CI.

---

## 4. Intelligent Edge Scheduling

Move beyond simple round-robin or least-connections:
- Cost-aware routing (prefer cheaper / lower-latency origins when equivalent).
- Congestion-aware routing using real-time signals from the probe mesh.
- ML-assisted (lightweight) prediction of best backend/POP for a given request signature.
- Integration with Programmable Edge for custom scheduling logic per domain.

---

## 5. Advanced Caching Strategies

### Multi-Level Plan Cache
- L1: In-process (compiler)
- L2: Local `dcpd` / edge agent (`redb` or memory-mapped)
- L3: Distributed (Dragonfly/Redis) with domain affinity
- Content-addressed + versioned for perfect invalidation.

### State Cache Warming
- Proactively warm Domain State Primitives for predicted high-traffic domains.
- Use access patterns from provenance to decide what to pre-load.

---

## 6. Chaos Engineering for Speed

You can't optimize what breaks under pressure.

- Regularly inject latency, packet loss, and POP failures into staging.
- Measure impact on p99 / p999 routing activation and tail latency.
- Automated regression detection when new techniques degrade tail behavior.

---

## Implementation Priority

| Technique | When to implement | Expected gain |
|-----------|-------------------|---------------|
| Predictive pre-warming | Phase 3 (Vertical) | High for repeat traffic |
| NUMA + hugepages | Phase 2–3 | Medium-High (hardware efficiency) |
| Tail latency hardening | Ongoing | Critical for production trust |
| Intelligent scheduling | Phase 3+ | High for complex tenants |
| Advanced caching | Phase 2 | High ROI |

---

## Related Documents

- [dcp-07-extreme-speed-optimizations.md](./dcp-07-extreme-speed-optimizations.md) — Foundational techniques
- [dcp-11-programmable-edge.md](../03-core-systems/dcp-11-programmable-edge.md) — Compute that benefits from these optimizations
- [dcp-12-domain-state-primitives.md](../03-core-systems/dcp-12-domain-state-primitives.md) — Stateful fast path

---

*These techniques turn "very fast" into "insanely predictable and efficient at global scale".*