# Domain Control Plane (DCP)

**Developer-native infrastructure for programmable, permissioned, versioned, observable, and reversible domains.**

> Like Resend for domains — not a DNS replacement. DNS remains the internet substrate. DCP compiles intent into safe operations across registrar state, authoritative DNS, TLS, email records, provider verification, route runtime, observability, policy, and rollback.

---

## Document Map

| Category | Path | Purpose |
|----------|------|---------|
| Conventions | [dcp-conventions.md](./dcp-conventions.md) | Naming, structure, and terminology standards |
| Glossary | [glossary/dcp-glossary.md](./glossary/dcp-glossary.md) | Canonical definitions |
| Vision | [01-vision/](./01-vision/) | Product thesis, category, principles, competitive positioning |
| Architecture | [02-architecture/](./02-architecture/) | System overview, layers, pipeline, deployment, speed, tech stack |
| Core Systems | [03-core-systems/](./03-core-systems/) | Ten engineering feats (kernel → AI planner) |
| APIs | [04-apis/](./04-apis/) | REST surface + OpenAPI reference |
| Schemas | [05-schemas/](./05-schemas/) | JSON/YAML schemas for all domain objects |
| Security | [06-security/](./06-security/) | Threat model, trust boundaries, controls |
| Roadmap | [07-roadmap/](./07-roadmap/) | MVP phases, research experiments, open questions |
| Examples | [08-examples/](./08-examples/) | SaaS, CI/CD, email, blue-green walkthroughs |
| Integrations | [09-integrations/](./09-integrations/) | Terraform, edge platforms, Kubernetes |
| Reference | [10-reference/](./10-reference/) | Error catalog, webhook payloads, decision log |

---

## Reading Order

### Executive / Product (45 min)

1. [dcp-01-product-thesis.md](./01-vision/dcp-01-product-thesis.md)
2. [dcp-02-category-definition.md](./01-vision/dcp-02-category-definition.md)
3. [dcp-04-honest-constraints.md](./01-vision/dcp-04-honest-constraints.md)
4. [dcp-05-competitive-positioning.md](./01-vision/dcp-05-competitive-positioning.md)
5. [dcp-01-system-overview.md](./02-architecture/dcp-01-system-overview.md)

### Engineering Deep Dive (5–7 hours)

6. [03-core-systems/](./03-core-systems/) — all ten documents in numeric order
7. [02-architecture/dcp-05-multi-tenant-saas-model.md](./02-architecture/dcp-05-multi-tenant-saas-model.md)
8. [04-apis/](./04-apis/) + [05-schemas/](./05-schemas/)
9. [06-security/](./06-security/)

### Implementation Patterns (2 hours)

10. [08-examples/](./08-examples/) — start with multi-tenant SaaS + blue-green
11. [09-integrations/](./09-integrations/) — match your stack
12. [10-reference/](./10-reference/) — errors and webhooks

### Planning & Research (1 hour)

13. [07-roadmap/](./07-roadmap/)
14. [10-reference/dcp-03-decision-log.md](./10-reference/dcp-03-decision-log.md)

---

## Core Thesis (One Paragraph)

DNS is globally distributed, eventually consistent, and registrar-bound — it cannot be changed instantly everywhere. **Routing behind stable DNS can be changed instantly for new requests.** DCP exploits this asymmetry: users declare *intent* (`api.example.com → service X, TLS auto, SPF aligned`); a deterministic compiler lowers intent to provider-specific operations; a transactional kernel applies them with leases, idempotency, and rollback; a route runtime serves traffic immediately while DNS propagates; and policy, provenance, and observability make every change auditable and reversible.

---

## The Ten Engineering Feats

| # | System | Doc |
|---|--------|-----|
| 1 | Transactional Domain Kernel | [dcp-01-transactional-domain-kernel.md](./03-core-systems/dcp-01-transactional-domain-kernel.md) |
| 2 | Domain Intent Compiler | [dcp-02-domain-intent-compiler.md](./03-core-systems/dcp-02-domain-intent-compiler.md) |
| 3 | Hosted & Self-Hosted Route Runtime | [dcp-03-route-runtime.md](./03-core-systems/dcp-03-route-runtime.md) |
| 4 | Domain OAuth & Scoped Capability Tokens | [dcp-04-domain-oauth-capability-tokens.md](./03-core-systems/dcp-04-domain-oauth-capability-tokens.md) |
| 5 | Signed Provider Recipe Runtime | [dcp-05-signed-provider-recipe-runtime.md](./03-core-systems/dcp-05-signed-provider-recipe-runtime.md) |
| 6 | Ownership & Provenance Graph | [dcp-06-ownership-provenance-graph.md](./03-core-systems/dcp-06-ownership-provenance-graph.md) |
| 7 | Certificate Firewall | [dcp-07-certificate-firewall.md](./03-core-systems/dcp-07-certificate-firewall.md) |
| 8 | Subdomain Takeover Immune System | [dcp-08-subdomain-takeover-immune-system.md](./03-core-systems/dcp-08-subdomain-takeover-immune-system.md) |
| 9 | Domain Tests, Replay, Simulation & Rollback | [dcp-09-tests-replay-simulation-rollback.md](./03-core-systems/dcp-09-tests-replay-simulation-rollback.md) |
| 10 | AI Planner / Debugger (Policy-Bound) | [dcp-10-ai-planner-debugger.md](./03-core-systems/dcp-10-ai-planner-debugger.md) |

---

## Quick Links by Role

| Role | Start here |
|------|------------|
| Founder / PM | [Product thesis](./01-vision/dcp-01-product-thesis.md) → [Competitive positioning](./01-vision/dcp-05-competitive-positioning.md) → [MVP roadmap](./07-roadmap/dcp-01-mvp-roadmap.md) |
| Platform engineer (SaaS) | [Multi-tenant model](./02-architecture/dcp-05-multi-tenant-saas-model.md) → [SaaS intent example](./08-examples/dcp-01-multi-tenant-saas-intent.md) |
| Infra / tech lead | [Technology stack](./02-architecture/dcp-08-technology-stack.md) → [Speed optimizations](./02-architecture/dcp-07-extreme-speed-optimizations.md) |
| SRE / DevOps | [CI/CD example](./08-examples/dcp-02-ci-cd-workflow.md) → [Error catalog](./10-reference/dcp-01-error-catalog.md) |
| Security | [Threat model](./06-security/dcp-01-threat-model.md) → [Certificate firewall](./03-core-systems/dcp-07-certificate-firewall.md) → [Takeover immune system](./03-core-systems/dcp-08-subdomain-takeover-immune-system.md) |
| API consumer | [API overview](./04-apis/dcp-01-api-overview.md) → [OpenAPI reference](./04-apis/dcp-07-openapi-reference.md) |

---

## Document Count

| Category | Files |
|----------|-------|
| Vision | 5 |
| Architecture | 8 |
| Core Systems | 10 |
| APIs | 7 |
| Schemas | 7 |
| Security | 3 |
| Roadmap | 3 |
| Examples | 4 |
| Integrations | 3 |
| Reference | 3 |
| Root + Glossary | 3 |
| **Total** | **56** |

---

## Status

| Attribute | Value |
|-----------|-------|
| Document version | `0.1.0-draft` |
| Last updated | 2026-06-28 |
| Maturity | Architecture & research specification |