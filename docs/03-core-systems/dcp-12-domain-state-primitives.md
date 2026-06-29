# Domain State Primitives

| Field | Value |
|-------|-------|
| Doc ID | `dcp-core-12` |
| Category | Core Systems |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-03, dcp-core-11, dcp-arch-07 |
| Created | 2026-06-28 |

---

## Summary

DCP provides **first-class, domain-attached state primitives** so that routes and programmable edge functions can read/write fast local state (KV, counters, rate limiters, sessions, feature flags) with strong consistency options and durability guarantees — inspired by Cloudflare Durable Objects / KV and Vercel Edge Config, but designed as native DCP concepts with full versioning, provenance, and multi-backend portability.

---

## Why This Matters

Static routing is table stakes. Real power comes from **stateful behavior at the edge** tied to domains:
- Rate limiting per API key or IP
- A/B testing & feature flags
- Session / personalization state
- Abuse detection counters
- Simple queues or durable execution hints

Without native state, teams are forced to call back to origin or external databases, killing the speed advantage.

---

## Core Primitives

| Primitive | Description | Latency target | Durability |
|-----------|-------------|----------------|------------|
| **KV** | Simple key-value per domain or route | < 1ms local | Configurable (memory → durable) |
| **Counters** | Atomic increment/decrement with reset policies | < 1ms | Durable on commit |
| **Rate Limiters** | Token bucket or sliding window per key | < 1ms | Durable |
| **Feature Flags** | Versioned flag evaluation with targeting | < 1ms | Durable + provenance |
| **Sessions** | Short-lived session store with TTL | < 2ms | Memory + durable fallback |

All primitives are **namespaced by domain** (or route) by default for isolation.

---

## Backend Strategy (Cloudflare + Portable)

**Hosted (Stack A):**
- Primary: Cloudflare Durable Objects / KV / R2 for durability
- Fast path: In-memory + Dragonfly/Redis at POPs with async durable write

**Self-hosted / Stack C:**
- Primary: etcd, NATS KV, or CockroachDB / PostgreSQL
- Fast path: In-memory + local Redis/Dragonfly

**Abstraction layer** in DCP ensures the same API works everywhere. The compiler and edge only see the DCP primitive interface.

---

## Integration with Programmable Edge

WASM functions (see dcp-11) get explicit allowlists:

```json
{
  "allowed_state": {
    "kv": ["user:preferences:*"],
    "counters": ["rate_limit:api_key:*"],
    "rate_limiters": ["global:login"]
  }
}
```

State access is mediated, audited, and versioned with the bundle that deployed the compute.

---

## Performance & Consistency

- Local POP reads/writes are sub-millisecond.
- Durable writes are async by default (fire-and-forget) with optional synchronous commit for critical operations.
- Strong consistency available within a POP; eventual across global POPs (with conflict resolution hooks).
- This keeps us on target for the extreme speed SLOs even with stateful compute.

---

## Related Documents

- [dcp-11-programmable-edge.md](./dcp-11-programmable-edge.md) — Compute that uses state
- [dcp-03-route-runtime.md](./dcp-03-route-runtime.md) — How state config lives in bundles
- [dcp-07-extreme-speed-optimizations.md](../02-architecture/dcp-07-extreme-speed-optimizations.md) — Latency targets

---

*Domain State Primitives give DCP the same "edge state superpowers" as Cloudflare and Vercel while staying portable and domain-native.*