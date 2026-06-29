# Observability API

| Field | Value |
|-------|-------|
| Doc ID | `dcp-api-05` |
| Category | APIs |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-01, dcp-api-01 |

---

## Probes

### Run Probe

```
POST /v1/domains/{domain}/probes/run
```

```json
{
  "targets": ["api.example.com"],
  "checks": ["dns", "tls", "http", "propagation"]
}
```

### Response

```json
{
  "data": {
    "probe_run_id": "probe_991",
    "results": [
      {
        "fqdn": "api.example.com",
        "dns": { "cname": "tenant.edge.dcp.dev", "match": true },
        "tls": { "valid": true, "expires_in_days": 74 },
        "http": { "status": 200, "latency_ms": 42 },
        "propagation": { "p50_complete": true, "p95_eta_minutes": 0 }
      }
    ]
  }
}
```

---

## Propagation Status

```
GET /v1/domains/{domain}/propagation?fqdn=api.example.com
```

```json
{
  "data": {
    "fqdn": "api.example.com",
    "authoritative": "confirmed",
    "global": {
      "p50": "complete",
      "p95": "propagating",
      "p95_eta_minutes": 18
    },
    "resolver_sample_size": 240,
    "last_updated": "2026-06-28T12:15:00Z"
  }
}
```

---

## Metrics Export

```
GET /v1/domains/{domain}/metrics?from=...&to=...
```

Prometheus-compatible endpoint (self-hosted):

```
GET /metrics
```

Key metrics documented in core system docs.

---

## Audit Log

```
GET /v1/orgs/{org_id}/audit?actor=app_ci&from=...
```

```json
{
  "data": [
    {
      "timestamp": "2026-06-28T12:00:00Z",
      "actor": "app_ci",
      "action": "transaction.committed",
      "resource": "domain:example.com",
      "transaction_id": "txn_7b2",
      "capability_token_id": "cap_991"
    }
  ]
}
```

---

## Real-Time Stream

```
GET /v1/domains/{domain}/events
Accept: text/event-stream
```

Streams probe updates, transaction phases, takeover alerts.