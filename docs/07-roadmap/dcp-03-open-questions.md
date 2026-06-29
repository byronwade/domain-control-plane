# Open Questions

| Field | Value |
|-------|-------|
| Doc ID | `dcp-roadmap-03` |
| Category | Roadmap |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | all core docs |

---

## Summary

Unresolved architectural and product decisions. Each question includes options, tradeoffs, and decision deadline.

---

## Category: Product & GTM

### OQ-01: Pricing unit

| Option | Pros | Cons |
|--------|------|------|
| Per domain | Simple | Penalizes multi-tenant SaaS |
| Per FQDN route | Aligns with value | Complex metering |
| Per transaction | Usage-based | Unpredictable bills |

**Decision needed by:** Phase 1 beta (Week 12)

---

### OQ-02: Default DNS strategy for hosted

| Option | Pros | Cons |
|--------|------|------|
| CNAME to proxy only | Minimal churn | Apex complexity |
| NS delegation to DCP adapter | Full control | High onboarding friction |
| Customer keeps DNS | Low friction | Limited immune visibility |

**Tentative:** CNAME/proxy default; NS optional.

---

## Category: Architecture

### OQ-03: Intent store: Git-native vs database-native

| Option | Pros | Cons |
|--------|------|------|
| Git per domain | Developer familiarity, PR flow | Scale? merge conflicts |
| Postgres JSONB | Transaction coupling | Less visible to devs |
| Hybrid | Git export + DB source of truth | Complexity |

**Decision needed by:** Phase 0 (Week 4)

---

### OQ-04: Event bus for kernel

| Option | Pros | Cons |
|--------|------|------|
| Postgres outbox | Simple ops | Throughput ceiling |
| Kafka/NATS | Scale, replay | Ops burden |
| Temporal/Cadence | Built-in sagas | Vendor lock |

**Tentative:** Postgres outbox MVP → NATS Phase 3.

---

### OQ-05: Route runtime implementation

| Option | Pros | Cons |
|--------|------|------|
| Envoy + custom xDS | Battle-tested | Config complexity |
| Rust proxy (pingora-like) | Performance | Build cost |
| Extend existing CDN (CF Workers) | Time to market | Neutrality perception |

**Decision needed by:** Phase 0 (Week 2)

---

## Category: Security

### OQ-06: Who can publish recipes?

| Option | Pros | Cons |
|--------|------|------|
| DCP official only (MVP) | Trust | Slow provider coverage |
| Vendor-signed | Quality | Partnership dependency |
| Community | Fast growth | Malware risk |

**Tentative:** Official MVP → vendor Phase 3 → community gated.

---

### OQ-07: Break-glass scope

Should break-glass grant full zone admin or maximum compensating rollback only?

**Security favors:** Rollback-only + registrar manual.

**Enterprise favors:** Time-limited full admin with ticket.

---

## Category: DNS Physics

### OQ-08: Wildcard apex on all providers

ALIAS/ANAME/CNAME flattening varies. Support matrix incomplete.

**Research:** R2 experiment outcomes inform UX fallbacks.

---

### OQ-09: IPv6-only origins

Dual-stack probe requirements? Runtime happy eyeballs?

**Tentative:** Dual-stack required for GA hosted runtime.

---

## Category: AI

### OQ-10: AI in critical path?

| Option | Pros | Cons |
|--------|------|------|
| Never (debugger only) | Safest | Less magic |
| Planner with mandatory human approve | Faster UX | Process friction |
| Auto-apply staging only | CI speed | Staging ≠ prod parity |

**Principle locked:** AI never bypasses compiler/policy (P3). Scope of automation TBD.

---

### OQ-11: Customer data in model training

Default opt-out vs opt-in for anonymized failure pattern learning.

**Tentative:** Opt-out default (GDPR-friendly).

---

## Category: Ecosystem

### OQ-12: Terraform relationship

Compete, complement, or acquire state?

| Stance | Implication |
|--------|-------------|
| Complement | `terraform-provider-dcp` exports intent |
| Replace | Migration tools from TF state |
| Sync bidirectional | High complexity |

**Tentative:** Complement Phase 4.

---

### OQ-13: ICANN / registrar partnerships

Direct registrar integrations vs aggregator (e.g., Domain Connect spec).

**Research:** Evaluate Domain Connect 2.0 coverage.

---

## Category: Legal & Compliance

### OQ-14: Liability for takeover false negatives

Insurance? SLA credits? Pure disclaimer?

Requires legal review before enterprise contracts.

---

### OQ-15: WHOIS / privacy regulation

Provenance may store claim evidence touching personal data.

**Action:** DPIA before EU GA.

---

## Decision Log Template

When resolved, move to `dcp-decision-log.md` (future):

```markdown
## DEC-NNN: {Title}
- **Date:** YYYY-MM-DD
- **Status:** accepted
- **Context:** ...
- **Decision:** ...
- **Consequences:** ...
```

---

## How to Contribute

Open questions welcome via issue template `open-question.md` with:

1. Problem statement
2. Options considered
3. Recommended experiment (if any)
4. Proposed owner