# Policy Schema

| Field | Value |
|-------|-------|
| Doc ID | `dcp-schema-04` |
| Category | Schemas |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-schema-01, dcp-api-04 |

---

## PolicyPack

```json
{
  "$id": "https://schemas.dcp.dev/v1/policy-pack.schema.json",
  "type": "object",
  "required": ["pack_id", "rules"],
  "properties": {
    "pack_id": { "type": "string" },
    "version": { "type": "string" },
    "rules": {
      "type": "object",
      "properties": {
        "cert_policy": { "$ref": "#/$defs/CertPolicy" },
        "takeover_policy": { "$ref": "#/$defs/TakeoverPolicy" },
        "raw_dns_policy": { "$ref": "#/$defs/RawDnsPolicy" },
        "capability_constraints": { "type": "array", "items": { "$ref": "#/$defs/CapabilityRule" } },
        "production_gates": { "$ref": "#/$defs/ProductionGates" }
      }
    }
  },
  "$defs": {
    "CertPolicy": {
      "type": "object",
      "properties": {
        "mode": { "enum": ["strict", "enterprise", "audit"] },
        "allow_wildcard": { "type": "boolean" },
        "max_sans": { "type": "integer" },
        "forbidden_patterns": { "type": "array", "items": { "type": "string" } }
      }
    },
    "TakeoverPolicy": {
      "type": "object",
      "properties": {
        "quarantine_threshold": { "type": "integer", "minimum": 0, "maximum": 100 },
        "auto_remove_dangling_cname": { "type": "boolean" },
        "scan_interval_seconds": { "type": "integer" }
      }
    },
    "RawDnsPolicy": {
      "type": "object",
      "properties": {
        "allow": { "type": "boolean" },
        "allowed_types": { "type": "array", "items": { "type": "string" } },
        "require_mfa": { "type": "boolean" }
      }
    },
    "ProductionGates": {
      "type": "object",
      "properties": {
        "require_ci_plan_hash": { "type": "boolean" },
        "require_human_approval_for_agents": { "type": "boolean" },
        "min_simulation_pass_rate": { "type": "number" }
      }
    }
  }
}
```

---

## Policy Decision Record

```json
{
  "decision_id": "pol_991",
  "plan_hash": "sha256:...",
  "decision": "allow",
  "evaluated_at": "2026-06-28T12:00:00Z",
  "rules": [
    { "rule_id": "cert.strict", "result": "pass" },
    { "rule_id": "takeover.score", "result": "pass", "detail": { "score": 12 } }
  ],
  "actor_id": "app_ci"
}
```

Stored immutably; referenced by kernel commit.