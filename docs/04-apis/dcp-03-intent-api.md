# Intent API

| Field | Value |
|-------|-------|
| Doc ID | `dcp-api-03` |
| Category | APIs |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-02, dcp-api-01 |

---

## Get Current Intent

```
GET /v1/domains/{domain}/intent
```

### Response

```json
{
  "data": {
    "apiVersion": "dcp.dev/v1",
    "kind": "DomainIntent",
    "metadata": {
      "domain": "example.com",
      "environment": "production",
      "version": "int_v13",
      "updated_at": "2026-06-28T12:00:00Z"
    },
    "spec": {
      "routes": [],
      "email": {},
      "verifications": [],
      "policy": {}
    }
  }
}
```

---

## List Intent Versions

```
GET /v1/domains/{domain}/intent/versions?limit=20
```

```json
{
  "data": [
    {
      "version": "int_v13",
      "content_hash": "sha256:...",
      "transaction_id": "txn_7b2",
      "provenance_id": "prov_f44",
      "created_at": "2026-06-28T12:00:00Z"
    }
  ]
}
```

---

## Get Specific Version

```
GET /v1/domains/{domain}/intent/versions/{version}
```

---

## Diff Versions

```
GET /v1/domains/{domain}/intent/diff?from=int_v12&to=int_v13
```

```json
{
  "data": {
    "from": "int_v12",
    "to": "int_v13",
    "changes": [
      {
        "path": "spec.routes[0].traffic.origin",
        "op": "replace",
        "old": "https://v1.internal",
        "new": "https://v2.internal"
      }
    ],
    "plan_preview_hash": "sha256:..."
  }
}
```

---

## Compile (No Transaction)

```
POST /v1/domains/{domain}/compile
```

```json
{
  "intent_delta": {},
  "base_version": "int_v13",
  "options": { "simulation": false }
}
```

### Response

```json
{
  "data": {
    "plan_hash": "sha256:8f3a...",
    "operations_count": 3,
    "dns_ops": 0,
    "routing_ops": 1,
    "policy_decision": "allow",
    "errors": []
  }
}
```

---

## Branching (Staging Overlay)

```
PUT /v1/domains/{domain}/intent/branches/staging
```

```json
{
  "overlay": {
    "routes": [
      { "host": "api.staging.example.com", "traffic": { "origin": "https://stg.internal" } }
    ]
  },
  "base_version": "int_v13"
}
```

Promote:

```
POST /v1/domains/{domain}/intent/branches/staging/promote
```

Creates transaction merging overlay → production.

---

## Import / Export

```
GET  /v1/domains/{domain}/intent/export?format=yaml
POST /v1/domains/{domain}/intent/import
```

Import validates schema; does not apply — returns proposed transaction.