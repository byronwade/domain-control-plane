# Decision Log

| Field | Value |
|-------|-------|
| Doc ID | `dcp-ref-03` |
| Category | Reference |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-roadmap-03 |

---

## Summary

Architectural decisions recorded as they are resolved. Open questions remain in [dcp-03-open-questions.md](../07-roadmap/dcp-03-open-questions.md).

---

## DEC-001: Stable DNS + Instant Route Split

| Field | Value |
|-------|-------|
| Date | 2026-06-28 |
| Status | accepted |
| Context | DNS cannot be globally instant; product must still feel instant |
| Decision | Separate `routing_status` from `dns_status`; optimize for route runtime behind stable CNAME |
| Consequences | Honest propagation UX; hosted runtime is core data plane investment |

---

## DEC-002: WASM Recipe Sandbox

| Field | Value |
|-------|-------|
| Date | 2026-06-28 |
| Status | accepted |
| Context | Provider SDK proliferation in kernel is unmaintainable |
| Decision | Signed WASM recipes with network allowlist; kernel invokes via Recipe Runtime |
| Consequences | R3 experiment validates perf; fallback to gRPC workers if needed |

---

## DEC-003: AI Proposal-Only

| Field | Value |
|-------|-------|
| Date | 2026-06-28 |
| Status | accepted |
| Context | AI agents will request domain changes |
| Decision | AI outputs `PlanProposal` only; must compile + pass policy before transaction |
| Consequences | Slower agent UX but zero bypass path; human-in-loop for production |

---

## DEC-004: Saga Not 2PC

| Field | Value |
|-------|-------|
| Date | 2026-06-28 |
| Status | accepted |
| Context | Registrars and DNS providers don't support distributed transactions |
| Decision | Kernel uses saga with compensations; explicit `failed` + escalation on compensation fail |
| Consequences | R6 chaos testing required before GA |

---

## DEC-005: MVP Provider Scope

| Field | Value |
|-------|-------|
| Date | 2026-06-28 |
| Status | accepted |
| Context | Recipe coverage vs time-to-market |
| Decision | MVP: Cloudflare + Route53 + Google Cloud DNS recipes only |
| Consequences | Customers on other DNS wait Phase 2; compiler returns `COMPILE_RECIPE_MISSING` clearly |

---

## DEC-006: Hosted Default Deployment

| Field | Value |
|-------|-------|
| Date | 2026-06-28 |
| Status | accepted |
| Context | Self-hosted is enterprise sales cycle |
| Decision | Default GA mode is hosted control plane + hosted runtime; hybrid/self-hosted Phase 2–3 |
| Consequences | Multi-tenant security bar is P0; data residency via self-hosted later |

---

## DEC-007: Phased Technology Stack (B → A)

| Field | Value |
|-------|-------|
| Date | 2026-06-28 |
| Status | accepted |
| Context | Need frameworks for speed-first domain control plane with transactional kernel |
| Decision | MVP on Stack B (Go + Connect + Temporal + Envoy Delta xDS + CoreDNS); GA speed tier migrates to Stack A (Rust Pingora + Hickory + wasmtime + Axum); Enterprise ships Stack C (Envoy Gateway + Helm) |
| Consequences | Hire Go first; add Rust for edge/DNS at Phase 2; kernel uses dual-path (Tokio fast path + Temporal full path) |

---

## Template for New Decisions

```markdown
## DEC-NNN: Title
| Field | Value |
|-------|-------|
| Date | YYYY-MM-DD |
| Status | proposed \| accepted \| superseded |
| Context | ... |
| Decision | ... |
| Consequences | ... |
| Supersedes | DEC-XXX (if any) |
```