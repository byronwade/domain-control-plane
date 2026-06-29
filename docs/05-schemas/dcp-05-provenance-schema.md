# Provenance Schema

| Field | Value |
|-------|-------|
| Doc ID | `dcp-schema-05` |
| Category | Schemas |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-schema-01, dcp-core-06 |

---

## ProvenanceRecord

```json
{
  "$id": "https://schemas.dcp.dev/v1/provenance-record.schema.json",
  "type": "object",
  "required": [
    "provenance_id", "intent_version", "transaction_id",
    "plan_hash", "actor", "timestamp", "prev_hash"
  ],
  "properties": {
    "provenance_id": { "type": "string", "pattern": "^prov_" },
    "intent_version": { "type": "string" },
    "transaction_id": { "type": "string" },
    "plan_hash": { "type": "string" },
    "parent_provenance": { "type": "string" },
    "prev_hash": { "type": "string", "description": "Hash chain link" },
    "actor": { "$ref": "#/$defs/Actor" },
    "timestamp": { "type": "string", "format": "date-time" },
    "effects": {
      "type": "array",
      "items": { "$ref": "#/$defs/Effect" }
    },
    "signature": { "type": "string" }
  },
  "$defs": {
    "Actor": {
      "type": "object",
      "required": ["type", "id"],
      "properties": {
        "type": { "enum": ["user", "application", "agent", "system"] },
        "id": { "type": "string" },
        "grant_id": { "type": "string" }
      }
    },
    "Effect": {
      "type": "object",
      "required": ["kind", "resource"],
      "properties": {
        "kind": { "enum": ["route", "dns", "tls", "email", "verification"] },
        "resource": { "type": "string" },
        "state_hash": { "type": "string" },
        "bundle_version": { "type": "integer" }
      }
    }
  }
}
```

---

## ZoneClaim

```json
{
  "claim_id": "claim_abc",
  "zone": "example.com",
  "claimant_org_id": "org_abc",
  "status": { "enum": ["pending", "active", "disputed", "revoked"] },
  "evidence": [
    { "type": "registrar_oauth", "provider": "namecheap", "linked_at": "..." },
    { "type": "dns_txt", "record": "_dcp-challenge.example.com", "verified_at": "..." }
  ],
  "fences": ["fqdn:example.com"]
}
```

---

## QuarantineNode

```json
{
  "quarantine_id": "q_09",
  "fqdn": "staging.api.example.com",
  "risk_score": 92,
  "opened_at": "2026-06-28T12:00:00Z",
  "signals": [{ "code": "DANGLING_CNAME" }],
  "opened_by": "system:immune",
  "override_capability": "takeover:override"
}
```