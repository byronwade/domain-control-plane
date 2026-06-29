# Programmable Edge (Revolutionary Domain Compute)

| Field | Value |
|-------|-------|
| Doc ID | `dcp-core-11` |
| Category | Core Systems |
| Status | draft |
| Version | 0.2.0-draft |
| Depends on | dcp-core-03, dcp-arch-07 |
| Last major update | 2026-06-28 |

---

## Summary

DCP makes **programmable compute at the domain level** feel native and revolutionary — not bolted on.

Instead of treating domains as simple routing targets with optional Workers, DCP treats domains as programmable units where routing, compute, and state are deeply integrated, versioned together, and instantly reversible.

This is one of the key differentiators that makes DCP feel like a new category rather than "domains + edge functions."

---

## Revolutionary Aspects

- Compute is versioned and rolled back together with routing and state.
- WASM modules are first-class citizens of the domain model.
- Safety (policy engine + provenance) applies to compute the same way it applies to routing.
- Developers can express intent at the domain level and have it compiled safely into executable edge logic.

---

## Integration with Revolutionary Speed

The Programmable Edge is designed to add minimal overhead to our aggressive speed targets. With Wizer pre-initialization and careful sandboxing, compute can feel nearly native to the fast path.

---

## Related Documents

- [dcp-07-extreme-speed-optimizations.md](../02-architecture/dcp-07-extreme-speed-optimizations.md)
- [dcp-12-domain-state-primitives.md](./dcp-12-domain-state-primitives.md)

---

*Version 0.2.0 — Strengthened revolutionary domain programmability positioning.*