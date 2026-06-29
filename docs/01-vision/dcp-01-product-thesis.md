# Product Thesis

| Field | Value |
|-------|-------|
| Doc ID | `dcp-vision-01` |
| Category | Vision |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | none |

---

## Summary

Domain Control Plane (DCP) is **Resend for domains**: a developer-native API layer that makes domain configuration as ergonomic as sending email through an API — without replacing DNS as the internet's substrate.

Developers declare **intent**. DCP compiles, permissions, versions, observes, and reverses changes across the full domain stack.

---

## The Problem

Today, "connecting a domain" fragments across:

| Layer | Pain |
|-------|------|
| Registrar | WHOIS, locks, transfers, nameserver delegation |
| DNS | Record types, TTLs, apex ALIAS hacks, split horizons |
| TLS | ACME, CAA, renewal, multi-host SAN management |
| Email | SPF, DKIM, DMARC alignment, provider-specific tokens |
| SaaS verification | `_acme-challenge`, `_github-challenge`, Google/M365 TXT |
| Routing | CDN, load balancers, serverless, blue/green |
| Security | Subdomain takeover, certificate mis-issuance, dangling CNAMEs |
| Operations | No rollback, no audit trail, no CI integration |

Each provider has different APIs, semantics, and failure modes. Teams encode tribal knowledge in runbooks, Terraform, and prayer.

---

## The Insight

Two clocks run in parallel:

| Clock | Speed | What changes |
|-------|-------|--------------|
| **DNS propagation** | Seconds to 48+ hours | What resolvers return |
| **Route runtime** | Milliseconds | Where new connections go |

**DCP separates stable DNS from dynamic routing.**

Point `api.example.com` once at a DCP route endpoint (or CNAME to hosted runtime). Subsequent path, origin, TLS, and traffic shifts happen **instantly for new requests** without waiting for global DNS convergence.

DNS is not replaced — it is **stabilized** and **abstracted**.

---

## Product Positioning

```
                    ┌─────────────────────────────────┐
                    │     Developer / Agent / CI      │
                    │   Intent API · OAuth · SDK    │
                    └───────────────┬─────────────────┘
                                    │
                    ┌───────────────▼─────────────────┐
                    │    DOMAIN CONTROL PLANE (DCP)   │
                    │  compile · permission · version │
                    │  observe · rollback · simulate  │
                    └───────────────┬─────────────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          ▼                         ▼                         ▼
   ┌─────────────┐          ┌─────────────┐          ┌─────────────┐
   │  Registrar  │          │  Auth DNS   │          │Route Runtime│
   │   Adapters  │          │  Providers  │          │ hosted/self │
   └─────────────┘          └─────────────┘          └─────────────┘
          │                         │                         │
          └─────────────────────────┴─────────────────────────┘
                                    │
                          Internet Substrate (DNS)
```

**Comparable analogies:**

| Product | Substrate | Control Plane |
|---------|-----------|---------------|
| Resend | SMTP | Email API |
| Stripe | Card networks | Payments API |
| Vercel | HTTP/origin | Deploy & routing API |
| **DCP** | DNS/registrars | Domain intent API |

---

## Target Users

1. **Platform engineers** — Multi-tenant SaaS with customer custom domains
2. **DevOps / SRE** — Safe, auditable domain changes in CI/CD
3. **Security teams** — Takeover prevention, cert governance, provenance
4. **AI agents** — Scoped tokens for domain tasks without root registrar access
5. **Indie developers** — One API call to "make `app.example.com` work"

---

## Value Propositions

| Capability | Outcome |
|------------|---------|
| Programmable | Intent objects in code, not zone file archaeology |
| Permissioned | OAuth scopes per subdomain, record class, environment |
| Versioned | Git-like history; diff, branch (staging), promote |
| Observable | Propagation probes, TLS checks, structured audit log |
| Reversible | Compensating transactions; one-click rollback |

---

## Non-Goals (v1)

- Replacing ICANN/registrar infrastructure
- Operating a global authoritative DNS network (initially: adapter to existing providers)
- Guaranteeing instant global DNS visibility
- Hiding DNS from operators who need raw record access (escape hatch always available)

---

## Success Metrics

| Metric | Target (Year 1) |
|--------|-----------------|
| Time to first routed subdomain | < 60 seconds (hosted runtime) |
| Transaction rollback success | > 99.5% for supported providers |
| Takeover incidents (customers) | Zero undetected > 1 hour |
| Mean time to diagnose domain issue | < 5 min with observability API |
| Compiler plan rejection (policy) | 100% deterministic; zero bypass |