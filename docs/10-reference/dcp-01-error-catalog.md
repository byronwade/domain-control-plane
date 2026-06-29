# Error Catalog

| Field | Value |
|-------|-------|
| Doc ID | `dcp-ref-01` |
| Category | Reference |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-api-01 |

---

## Summary

Canonical error codes returned by DCP APIs. Each code maps to HTTP status, retry guidance, and documentation URL.

---

## Compile Errors (4xx)

| Code | HTTP | Retry | Meaning |
|------|------|-------|---------|
| `COMPILE_SCHEMA` | 400 | No | Intent fails JSON schema |
| `COMPILE_CONFLICT` | 400 | No | Overlapping routes, illegal DNS |
| `COMPILE_UNSUPPORTED` | 400 | No | Facet/provider combo not supported |
| `COMPILE_FENCE_VIOLATION` | 403 | No | Cross-tenant or undelegated FQDN |
| `COMPILE_CAPABILITY_DENIED` | 403 | No | Token scope insufficient |
| `COMPILE_PROVIDER_DRIFT` | 409 | Yes | Snapshot stale; refresh and retry |
| `COMPILE_RECIPE_MISSING` | 422 | No | No recipe for provider@operation |

---

## Policy Errors (422)

| Code | HTTP | Retry | Meaning |
|------|------|-------|---------|
| `POLICY_DENIED` | 422 | No | Plan failed policy rules |
| `POLICY_CERT_FIREWALL` | 422 | No | TLS blocked by firewall |
| `POLICY_RAW_DNS_DENIED` | 422 | No | Raw DNS escape hatch blocked |
| `POLICY_PRODUCTION_GATE` | 422 | No | Missing CI plan hash or approval |
| `POLICY_TAKEOVER_RISK` | 422 | No | Immune system blocked change |

---

## Transaction Errors

| Code | HTTP | Retry | Meaning |
|------|------|-------|---------|
| `TXN_IDEMPOTENCY_CONFLICT` | 409 | No | Same key, different payload |
| `TXN_LEASE_DENIED` | 409 | Yes | Another transaction holds lease |
| `TXN_LEASE_EXPIRED` | 500 | Manual | Lease TTL exceeded mid-apply |
| `TXN_OP_FAILED` | 500 | Partial | Provider op failed; may auto-rollback |
| `TXN_VERIFY_FAILED` | 500 | Partial | Post-apply verification failed |
| `TXN_COMPENSATION_FAILED` | 500 | Manual | Rollback incomplete — escalate |
| `TXN_PROVIDER_TIMEOUT` | 503 | Yes | Provider API timeout |
| `TXN_PROVIDER_RATE_LIMIT` | 429 | Yes | Backoff per Retry-After |

---

## Auth Errors (401/403)

| Code | HTTP | Retry | Meaning |
|------|------|-------|---------|
| `AUTH_MISSING` | 401 | No | No token provided |
| `AUTH_INVALID` | 401 | No | Token malformed or bad signature |
| `AUTH_EXPIRED` | 401 | Refresh | Token expired |
| `AUTH_REVOKED` | 403 | No | Capability revoked |
| `AUTH_ORG_MISMATCH` | 403 | No | Dcp-Org-Id doesn't match token |

---

## Runtime Errors

| Code | HTTP | Retry | Meaning |
|------|------|-------|---------|
| `RUNTIME_BUNDLE_INVALID` | 500 | No | Signature verification failed |
| `RUNTIME_BACKEND_UNHEALTHY` | 503 | Yes | All backends down |
| `RUNTIME_NOT_REGISTERED` | 403 | No | Self-hosted runtime not enrolled |

---

## AI Errors

| Code | HTTP | Retry | Meaning |
|------|------|-------|---------|
| `AI_PROPOSAL_INVALID` | 400 | Yes | PlanProposal schema fail |
| `AI_CONFIDENCE_LOW` | 422 | Human | Below auto-suggest threshold |
| `AI_DISABLED` | 403 | No | Org policy disables AI |

---

## Error Response Example

```json
{
  "error": {
    "code": "POLICY_CERT_FIREWALL",
    "message": "Certificate issuance blocked: dangling CNAME detected",
    "details": [
      {
        "field": "spec.routes[0].host",
        "reason": "takeover_risk_score=94",
        "fqdn": "staging.api.example.com"
      }
    ],
    "doc_url": "https://docs.dcp.dev/errors/POLICY_CERT_FIREWALL",
    "request_id": "req_abc",
    "remediation": {
      "suggested_transaction": null,
      "support_action": "Remove dangling CNAME or override quarantine"
    }
  }
}
```

---

## Retry Strategy Guide

| Code pattern | Client behavior |
|--------------|-----------------|
| `4xx` (except 429) | Fix request; do not retry |
| `429`, `503` | Exponential backoff + jitter |
| `TXN_LEASE_DENIED` | Retry after 2–5s |
| `TXN_IDEMPOTENCY_CONFLICT` | GET original transaction by key |