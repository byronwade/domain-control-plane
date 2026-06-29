# Transaction API

| Field | Value |
|-------|-------|
| Doc ID | `dcp-api-02` |
| Category | APIs |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-01, dcp-api-01 |

---

## Submit Transaction

```
POST /v1/domains/{domain}/transactions
```

### Request

```json
{
  "intent_delta": {
    "routes": [
      {
        "host": "api.example.com",
        "traffic": { "origin": "https://v2.internal" }
      }
    ]
  },
  "base_version": "int_v12",
  "options": {
    "auto_rollback": true,
    "wait_for": ["routing"],
    "dry_run": false
  }
}
```

### Response `201`

```json
{
  "data": {
    "transaction_id": "txn_7b2",
    "phase": "planning",
    "idempotency_key": "deploy-v2-20260628",
    "plan_hash": null,
    "status": {
      "routing": "pending",
      "dns": "pending",
      "tls": "pending"
    },
    "links": {
      "self": "/v1/transactions/txn_7b2",
      "events": "/v1/transactions/txn_7b2/events"
    }
  }
}
```

---

## Get Transaction

```
GET /v1/transactions/{txn_id}
```

### Response `200` (committed)

```json
{
  "data": {
    "transaction_id": "txn_7b2",
    "phase": "committed",
    "plan_hash": "sha256:8f3a...",
    "intent_version": "int_v13",
    "provenance_id": "prov_f44",
    "status": {
      "routing": "active",
      "dns": "propagating",
      "tls": "active"
    },
    "propagation": {
      "dns_p50_eta_minutes": 12,
      "dns_p95_eta_minutes": 45,
      "authoritative_confirmed_at": "2026-06-28T12:00:30Z"
    },
    "timing": {
      "routing_active_ms": 890,
      "total_ms": 4500
    }
  }
}
```

---

## List Transactions

```
GET /v1/domains/{domain}/transactions?phase=committed&limit=50
```

---

## Transaction Events (SSE)

```
GET /v1/transactions/{txn_id}/events
Accept: text/event-stream
```

```
event: phase_changed
data: {"phase":"applying","at":"2026-06-28T12:00:01Z"}

event: op_completed
data: {"op_id":"op_17","status":"success"}

event: committed
data: {"intent_version":"int_v13","routing_active_ms":890}
```

---

## Rollback

```
POST /v1/transactions/{txn_id}/rollback
```

### Request

```json
{
  "mode": "transaction",
  "options": {
    "wait_for": ["routing"]
  }
}
```

### Response `202`

```json
{
  "data": {
    "rollback_transaction_id": "txn_7b3",
    "phase": "planning",
    "restores_version": "int_v12"
  }
}
```

---

## Dry Run (Plan Only)

```
POST /v1/domains/{domain}/transactions
```

```json
{
  "intent_delta": { },
  "options": { "dry_run": true }
}
```

Returns `plan_hash`, operations list, policy decisions — no leases acquired.

---

## Wait Semantics

`options.wait_for`:

| Value | Blocks until |
|-------|--------------|
| `routing` | Route runtime active |
| `authoritative_dns` | NS confirm |
| `propagation_p50` | Probe network median |
| `committed` | Transaction committed (default) |

Long waits use SSE or webhook; HTTP sync max 120s.