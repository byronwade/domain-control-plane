# Domain State Primitives (Revolutionary Domain-Level State)

| Field | Value |
|-------|-------|
| Doc ID | `dcp-core-12` |
| Category | Core Systems |
| Status | draft |
| Version | 0.2.0-draft |
| Depends on | dcp-core-03, dcp-core-11 |
| Last major update | 2026-06-28 |

---

## Summary

DCP introduces **domain-level state as a first-class, programmable primitive** — deeply integrated with routing and compute.

This goes beyond simple KV stores. Domain State Primitives (KV, counters, rate limiters, feature flags, sessions) are versioned with the domain, rolled back together with routing changes, and accessible from both the fast path and Programmable Edge with strong safety guarantees.

This is a key part of what makes domain programming in DCP feel revolutionary rather than incremental.

---

## Revolutionary Value

- State lives at the domain level, not as a separate concern.
- State changes can be versioned and rolled back atomically with routing.
- Tight integration with Programmable Edge enables powerful patterns (stateful A/B testing, rate limiting, personalization) without leaving the fast path.
- Safety and provenance apply to state the same way they apply to routing and compute.

---

## Related Documents

- [dcp-11-programmable-edge.md](./dcp-11-programmable-edge.md)
- [dcp-07-extreme-speed-optimizations.md](../02-architecture/dcp-07-extreme-speed-optimizations.md)

---

*Version 0.2.0 — Strengthened revolutionary domain state positioning.*