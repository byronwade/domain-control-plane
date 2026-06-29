# Honest Constraints

| Field | Value |
|-------|-------|
| Doc ID | `dcp-vision-04` |
| Category | Vision |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-vision-01, dcp-vision-03 |

---

## Summary

DCP is visionary but physically honest. This document states hard limits so architects and customers set correct expectations.

---

## The Fundamental Asymmetry

```
┌──────────────────────────────────────────────────────────────────┐
│  CAN change instantly (new requests)                           │
│  • Route runtime origin / path / header rules                  │
│  • TLS cert selection at edge (already issued)                 │
│  • Traffic split weights at proxy                              │
│  • Feature flags on domain behavior                            │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│  CANNOT change instantly (global)                                │
│  • All resolver caches worldwide                                 │
│  • Registrar WHOIS propagation                                   │
│  • Third-party verifier cache (Google, Microsoft)                │
│  • CA OCSP/CT log visibility                                   │
│  • Email receiver SPF/DKIM cache                                 │
└──────────────────────────────────────────────────────────────────┘
```

**DCP exploits the first bucket to deliver instant developer experience while being truthful about the second.**

---

## DNS Propagation Reality

| Factor | Typical Range |
|--------|---------------|
| Authoritative update | 1–30 seconds (provider-dependent) |
| TTL-bound cache expiry | 300s – 86400s (operator-chosen) |
| Stale resolver behavior | Unpredictable; some ignore TTL |
| Global 95th percentile | Minutes to hours |
| Full convergence | Up to 48 hours (edge cases) |

**DCP behavior:**

- Returns probabilistic `propagation_horizon` from probe network
- Does not mark DNS `committed` until authoritative confirms
- Separates `routing_status: active` from `dns_status: propagating`

---

## Registrar Constraints

| Operation | Typical Duration | Reversible |
|-----------|------------------|------------|
| Nameserver change | 1–48 hours | Yes, with propagation delay |
| Domain transfer | 5–7 days | No mid-flight cancel |
| Registrar lock | Instant | Instant |
| Contact/WHOIS update | Minutes–hours | Partial |

**DCP behavior:** Registrar operations run in long-running transaction phase with explicit `pending_registrar` state.

---

## TLS / ACME Constraints

| Constraint | Detail |
|------------|--------|
| HTTP-01 / DNS-01 timing | CA retries; authorizations expire (~7 days) |
| CAA propagation | Must be visible before CA issues |
| Rate limits | Let's Encrypt per-domain weekly caps |
| Certificate transparency | Irreversible public log entry |

**DCP behavior:** Certificate firewall pre-checks CAA and takeover risk. Failed issuance triggers rollback of *intent*, not deletion of CT logs.

---

## Email Authentication Constraints

| Record | Cache behavior |
|--------|----------------|
| SPF | Receivers cache DNS; void lookups matter |
| DKIM | Selector rotation needs overlap period |
| DMARC | Aggregate reports lag 24–48 hours |

**DCP behavior:** Email intent includes `rotation_overlap_period`. Compiler emits dual-record windows.

---

## What "Instant" Means in DCP

| User action | Instant? | Mechanism |
|-------------|----------|-----------|
| Switch API origin v1 → v2 | **Yes** (new connections) | Route runtime |
| Add staging subdomain | **Mostly** | Wildcard/proxy pattern if pre-provisioned |
| Change apex A record | **No** (global DNS) | Standard propagation |
| Rollback route config | **Yes** | Runtime version revert |
| Rollback DNS intent | **Correct immediately at authoritative** | Global caches lag |
| Revoke capability token | **Yes** | Control plane state |
| Undo domain transfer | **No** | Registrar policy |

---

## SLA Framing (Recommended Customer Language)

> **Routing SLA:** 99.99% — changes effective for new requests within 5 seconds of commit.
>
> **DNS SLA:** Best-effort propagation visibility; authoritative update within 60 seconds; global probe median < 15 minutes for TTL ≤ 300s.
>
> **Security SLA:** Takeover detection < 5 minutes; certificate policy violation blocked 100%.

---

## Research Frontiers (Not Promises)

See [dcp-03-research-experiments.md](../07-roadmap/dcp-03-research-experiments.md):

- Split-horizon intent for internal vs external resolvers
- DNS prefetch signaling to major resolver networks (partnership-dependent)
- Intent-based anycast steering without record churn