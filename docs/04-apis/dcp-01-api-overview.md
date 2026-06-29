# API Overview

| Field | Value |
|-------|-------|
| Doc ID | `dcp-api-01` |
| Category | APIs |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-01 |

---

## Summary

DCP exposes a **REST-first** API (JSON) with optional gRPC for high-throughput runtime and kernel internal paths. All mutations flow through transactions.

---

## Base URLs

| Deployment | Base URL |
|------------|----------|
| Hosted | `https://api.dcp.dev/v1` |
| Self-hosted | `https://{customer-dcp}/v1` |

---

## Authentication

```
Authorization: Bearer {capability_jwt}
```

Or API key (server-to-server, maps to narrow capability):

```
Authorization: Dcp-Api-Key dcp_live_...
```

See [dcp-06-auth-api.md](./dcp-06-auth-api.md).

---

## Common Headers

| Header | Required | Purpose |
|--------|----------|---------|
| `Authorization` | Yes | Capability token |
| `Idempotency-Key` | Mutations | Transaction dedup |
| `Dcp-Org-Id` | Yes | Tenant namespace |
| `Dcp-Environment` | Optional | `production` / `staging` |
| `Dcp-Request-Id` | Optional | Tracing |

---

## Resource Model

```
/v1/orgs/{org_id}
/v1/domains/{domain}
/v1/domains/{domain}/intent
/v1/domains/{domain}/intent/versions
/v1/domains/{domain}/transactions
/v1/transactions/{txn_id}
/v1/domains/{domain}/routes
/v1/domains/{domain}/probes
/v1/domains/{domain}/takeover-risks
/v1/domains/{domain}/tests
/v1/simulations
/v1/oauth/authorize
/v1/oauth/token
/v1/capabilities/revoke
/v1/runtimes
/v1/ai/plan
/v1/ai/debug
```

---

## Standard Response Envelope

```json
{
  "data": {},
  "meta": {
    "request_id": "req_abc",
    "api_version": "v1"
  }
}
```

---

## Error Format

```json
{
  "error": {
    "code": "COMPILE_CONFLICT",
    "message": "Overlapping route on api.example.com",
    "details": [
      { "field": "spec.routes[0].host", "reason": "duplicate" }
    ],
    "doc_url": "https://docs.dcp.dev/errors/COMPILE_CONFLICT",
    "request_id": "req_abc"
  }
}
```

| HTTP | Meaning |
|------|---------|
| 400 | Invalid request / compile error |
| 401 | Missing auth |
| 403 | Capability denied |
| 404 | Resource not found |
| 409 | Idempotency conflict / lease denied |
| 422 | Policy rejected |
| 429 | Rate limited |
| 503 | Provider or kernel overloaded |

---

## Pagination

```
GET /v1/domains/example.com/transactions?limit=20&cursor=txn_xyz
```

```json
{
  "data": [],
  "pagination": {
    "next_cursor": "txn_abc",
    "has_more": true
  }
}
```

---

## Webhooks

| Event | Trigger |
|-------|---------|
| `transaction.phase_changed` | Any phase transition |
| `transaction.committed` | Success |
| `transaction.rolled_back` | Rollback complete |
| `probe.threshold_crossed` | Propagation milestone |
| `takeover.risk_detected` | Score > threshold |
| `cert_firewall.denied` | TLS blocked |
| `drift.detected` | Intent ≠ observed |

Delivery: signed HMAC-SHA256, retry 72h exponential backoff.

---

## SDKs (Planned)

| Language | Package |
|----------|---------|
| TypeScript | `@dcp/sdk` |
| Python | `dcp-python` |
| Go | `github.com/dcp-dev/sdk-go` |
| Terraform | `terraform-provider-dcp` |

---

## Rate Limits

| Tier | Write req/min | Read req/min |
|------|---------------|--------------|
| Free | 30 | 300 |
| Pro | 300 | 3000 |
| Enterprise | Custom | Custom |

Transaction submits count as 1 write regardless of op count.