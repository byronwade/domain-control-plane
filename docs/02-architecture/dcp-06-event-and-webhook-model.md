# Event and Webhook Model

| Field | Value |
|-------|-------|
| Doc ID | `dcp-arch-06` |
| Category | Architecture |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-api-01, dcp-core-01 |

---

## Summary

DCP emits **CloudEvents 1.0** over webhooks, SSE, and optional message bus (enterprise). Events are the observability nervous system connecting kernel, runtime, immune system, and customer automation.

---

## Event Envelope

```json
{
  "specversion": "1.0",
  "id": "evt_8f3a2b1c",
  "source": "dcp.dev/kernel",
  "type": "dcp.transaction.committed",
  "subject": "domain/example.com/txn/txn_7b2",
  "time": "2026-06-28T12:00:00Z",
  "datacontenttype": "application/json",
  "data": {
    "org_id": "org_abc",
    "domain": "example.com",
    "transaction_id": "txn_7b2",
    "intent_version": "int_v13",
    "status": {
      "routing": "active",
      "dns": "propagating"
    }
  }
}
```

---

## Event Catalog

| Type | Source | Trigger |
|------|--------|---------|
| `dcp.transaction.phase_changed` | kernel | Any phase transition |
| `dcp.transaction.op_completed` | kernel | Single op success/fail |
| `dcp.transaction.committed` | kernel | Successful commit |
| `dcp.transaction.rolled_back` | kernel | Rollback terminal |
| `dcp.transaction.failed` | kernel | Terminal failure |
| `dcp.intent.version_created` | api | New intent version pointer |
| `dcp.compile.plan_rejected` | compiler | Policy or validation fail |
| `dcp.probe.result` | observability | Probe run complete |
| `dcp.propagation.threshold_crossed` | observability | p50/p95 milestone |
| `dcp.route.bundle_activated` | runtime | New bundle live |
| `dcp.route.backend_unhealthy` | runtime | Health check fail |
| `dcp.cert_firewall.allowed` | cert-firewall | TLS approved |
| `dcp.cert_firewall.denied` | cert-firewall | TLS blocked |
| `dcp.cert.issued` | cert-firewall | Cert obtained |
| `dcp.cert.renewal_due` | cert-firewall | T-30d renewal |
| `dcp.takeover.risk_detected` | immune | Score > warn threshold |
| `dcp.takeover.quarantine_applied` | immune | FQDN quarantined |
| `dcp.drift.detected` | provenance | Intent ≠ observed |
| `dcp.capability.revoked` | auth | Token revoked |

Full payloads: [dcp-02-webhook-event-catalog.md](../10-reference/dcp-02-webhook-event-catalog.md).

---

## Webhook Delivery

### Registration

```
POST /v1/orgs/{org_id}/webhooks
```

```json
{
  "url": "https://hooks.example.com/dcp",
  "events": ["dcp.transaction.committed", "dcp.takeover.risk_detected"],
  "secret": "whsec_...",
  "domains": ["example.com"]
}
```

### Signature

```
DCP-Webhook-Signature: t=1719590400,v1=hmac_sha256_hex
```

Signed payload: `{timestamp}.{raw_body}`

### Retry Policy

| Attempt | Delay |
|---------|-------|
| 1 | immediate |
| 2 | 1 min |
| 3 | 5 min |
| 4–10 | exponential to 1 hour |
| 11+ | dead letter (72h retention) |

Success: HTTP 2xx within 30s.

---

## Server-Sent Events

### Transaction stream

```
GET /v1/transactions/{txn_id}/events
Accept: text/event-stream
```

### Domain stream

```
GET /v1/domains/{domain}/events
Accept: text/event-stream
```

Filter: `?types=dcp.takeover.risk_detected,dcp.transaction.committed`

---

## Enterprise Message Bus

Self-hosted / enterprise hosted option:

| Bus | Topic pattern |
|-----|---------------|
| Kafka | `dcp.{org_id}.{event_type}` |
| NATS | `dcp.events.{org_id}.{type}` |
| SQS | Per-org queue with filter policy |

Ordering: per `domain` partition key guaranteed.

---

## Idempotency

Consumers must dedupe on `event.id`. Retries reuse same `id`.

---

## PII & Redaction

| Field | Webhook |
|-------|---------|
| Actor email | Redacted → `actor.id` only |
| Origin URLs | Included |
| Provider credentials | Never included |
| WHOIS contacts | Never included |

---

## Integration Patterns

| Pattern | Events subscribed |
|---------|-------------------|
| CI gate unlock | `dcp.transaction.committed` |
| PagerDuty | `dcp.takeover.quarantine_applied`, `dcp.transaction.failed` |
| Slack deploy bot | `dcp.route.bundle_activated` |
| SIEM | All `dcp.*` types |
| Tenant webhook fanout | Platform filters by `data.actor.org_id` |