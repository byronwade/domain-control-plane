# Transaction Schema

| Field | Value |
|-------|-------|
| Doc ID | `dcp-schema-03` |
| Category | Schemas |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-schema-01, dcp-core-01 |

---

## Transaction Resource

```json
{
  "$id": "https://schemas.dcp.dev/v1/transaction.schema.json",
  "type": "object",
  "required": ["transaction_id", "domain", "phase", "org_id"],
  "properties": {
    "transaction_id": { "type": "string", "pattern": "^txn_" },
    "domain": { "type": "string", "format": "hostname" },
    "org_id": { "type": "string" },
    "phase": {
      "enum": [
        "pending", "planning", "planned", "leasing", "leased",
        "applying", "verifying", "committed",
        "rolling_back", "rolled_back", "failed"
      ]
    },
    "idempotency_key": { "type": "string", "maxLength": 128 },
    "plan_hash": { "type": "string", "pattern": "^sha256:" },
    "intent_version": { "type": "string" },
    "base_version": { "type": "string" },
    "provenance_id": { "type": "string" },
    "actor": { "$ref": "#/$defs/Actor" },
    "status": { "$ref": "#/$defs/FacetStatus" },
    "propagation": { "$ref": "#/$defs/Propagation" },
    "timing": { "type": "object" },
    "error": { "$ref": "#/$defs/TransactionError" }
  }
}
```

---

## Operation Plan

```json
{
  "$id": "https://schemas.dcp.dev/v1/operation-plan.schema.json",
  "type": "object",
  "required": ["plan_id", "plan_hash", "operations"],
  "properties": {
    "plan_id": { "type": "string" },
    "plan_hash": { "type": "string" },
    "compiler_version": { "type": "string" },
    "recipe_set_hash": { "type": "string" },
    "snapshot_id": { "type": "string" },
    "operations": {
      "type": "array",
      "items": { "$ref": "#/$defs/Operation" }
    },
    "policy_decision_id": { "type": "string" }
  },
  "$defs": {
    "Operation": {
      "type": "object",
      "required": ["op_id", "provider", "action", "params"],
      "properties": {
        "op_id": { "type": "string" },
        "provider": { "type": "string" },
        "action": { "type": "string" },
        "resource": { "type": "string" },
        "params": { "type": "object" },
        "depends_on": { "type": "array", "items": { "type": "string" } },
        "timeout_ms": { "type": "integer" },
        "compensation": { "type": "object" }
      }
    }
  }
}
```

---

## Facet Status

```json
{
  "routing": { "enum": ["pending", "active", "failed", "quarantined"] },
  "dns": { "enum": ["pending", "authoritative", "propagating", "complete", "failed"] },
  "tls": { "enum": ["pending", "active", "failed", "blocked"] },
  "email": { "enum": ["pending", "active", "failed"] }
}
```

Independent status per honest constraints doc.

---

## PlanProposal (AI)

```json
{
  "$id": "https://schemas.dcp.dev/v1/plan-proposal.schema.json",
  "type": "object",
  "required": ["kind", "proposal_id", "proposed_intent_delta"],
  "properties": {
    "kind": { "const": "PlanProposal" },
    "proposal_id": { "type": "string" },
    "proposed_intent_delta": { "type": "object" },
    "confidence": { "type": "number", "minimum": 0, "maximum": 1 },
    "required_capabilities": { "type": "array", "items": { "type": "string" } }
  }
}
```

**Not submittable as transaction** — must compile first.