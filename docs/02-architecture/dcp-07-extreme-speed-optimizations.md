# Extreme Speed Optimizations

| Field | Value |
|-------|-------|
| Doc ID | `dcp-arch-07` |
| Category | Architecture |
| Status | draft |
| Version | 0.3.0-draft |
| Depends on | dcp-vision-04, dcp-core-03, dcp-arch-08, dcp-core-01 |
| Last major update | 2026-06-28 |

---

## Summary

**DCP aims to deliver revolutionary speed** — not just incremental improvements. This document defines the techniques to make domain-level changes feel truly instant while maintaining strong safety and reversibility.

We exploit the fundamental asymmetry of the internet: DNS is slow and eventually consistent, but *routing, compute, and state behind stable DNS* can be changed instantly. DCP makes this asymmetry the default experience for developers.

Our goal is to remove entire classes of friction that developers currently accept as normal (DNS propagation, slow/unsafe rollbacks, unpredictable tail latency) and replace them with sub-second, predictable, and programmable domain operations.

---

## Revolutionary Speed Targets

| Priority | Layer | Target (p99) | Revolutionary Impact |
|----------|-------|--------------|----------------------|
| P0 | CLI/API response | < 80ms | Feels instantaneous locally |
| P0 | Routing active after change | **< 200ms** | Removes DNS propagation pain entirely for most operations |
| P0 | Rollback routing | **< 200ms** | True instant, atomic rollbacks become the norm |
| P1 | Authoritative DNS | < 150ms | Fast and correct when DNS is needed |
| P1 | Global propagation p95 | < 45s | Still excellent when changes must touch DNS |

These targets position DCP as noticeably faster than traditional IaC + CDN combinations and competitive with (or better than) the best edge platforms for domain-level operations.

---

## P0: DNS-free hot path (Core Revolutionary Lever)

After onboarding, **zero DNS operations** for routing, TLS, policy, or compute changes.

This is the single biggest lever for revolutionary speed. Most domain changes should feel instant because they never touch the slow DNS layer.

### Key Techniques

- Stable anycast entrypoints + pre-provisioned wildcards
- `routing_only: true` compiler path that completely bypasses DNS/registrar steps
- Fast Tokio kernel path (no Temporal) for pure routing changes
- Atomic per-FQDN version pointer swaps with quorum acknowledgment

**Result**: Developers experience domain changes as near-instant, removing one of the most painful parts of traditional infrastructure workflows.

---

## P0: Edge Delta Push + Version Pointers

We push small deltas and use atomic version pointer swaps rather than full config reloads. This is a key part of what makes DCP feel revolutionary compared to generic edge platforms.

Combined with Pingora’s performance, this approach allows us to achieve extremely low activation times while still supporting rich programmable edge and state features.

---

## Advanced Techniques for Revolutionary Feel

- **Predictive pre-warming**: Use provenance and access patterns to proactively warm routes, WASM modules, and state at likely POPs.
- **Hardware-aware execution**: NUMA awareness, deep io_uring integration, and zero-copy paths where possible.
- **Strong tail latency focus**: Not just good p99, but consistent, predictable performance globally.
- **Dual-path kernel**: Extremely fast path for common operations while the full path guarantees safety.

These techniques, layered on top of Pingora and Cloudflare’s network, are what allow DCP to feel revolutionary rather than just "fast."

---

## SLO Table (Revolutionary Positioning)

| Operation | Target (p99) | Why Revolutionary |
|-----------|--------------|-------------------|
| Routing active after intent | < 200ms | Removes DNS propagation from daily workflow |
| Instant rollback | < 200ms | Developers can ship fearlessly |
| `dcp ls` / status | < 50ms | Local-first, zero-friction experience |
| Programmable edge execution | Very low overhead | Compute feels native to the domain |

---

## Related Documents

- [dcp-08-technology-stack.md](./dcp-08-technology-stack.md)
- [dcp-11-programmable-edge.md](../03-core-systems/dcp-11-programmable-edge.md)
- [dcp-12-domain-state-primitives.md](../03-core-systems/dcp-12-domain-state-primitives.md)

---

*Version 0.3.0 — Refined with stronger revolutionary speed vision and tighter targets.*