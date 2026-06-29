# Intent Schema

| Field | Value |
|-------|-------|
| Doc ID | `dcp-schema-02` |
| Category | Schemas |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-schema-01, dcp-core-02 |

---

## DomainIntent

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.dcp.dev/v1/domain-intent.schema.json",
  "type": "object",
  "required": ["apiVersion", "kind", "metadata", "spec"],
  "properties": {
    "apiVersion": { "const": "dcp.dev/v1" },
    "kind": { "const": "DomainIntent" },
    "metadata": {
      "type": "object",
      "required": ["domain"],
      "properties": {
        "domain": { "type": "string", "format": "hostname" },
        "environment": { "enum": ["production", "staging", "development"] },
        "version": { "type": "string", "pattern": "^int_v[0-9]+$" },
        "labels": { "type": "object", "additionalProperties": { "type": "string" } }
      }
    },
    "spec": {
      "type": "object",
      "properties": {
        "routes": { "type": "array", "items": { "$ref": "#/$defs/Route" } },
        "email": { "$ref": "#/$defs/EmailAuth" },
        "verifications": { "type": "array", "items": { "$ref": "#/$defs/Verification" } },
        "tls": { "$ref": "#/$defs/TlsPolicy" },
        "policy": { "$ref": "#/$defs/DomainPolicyOverlay" },
        "tests": { "type": "array", "items": { "$ref": "#/$defs/DomainTest" } },
        "raw": {
          "type": "array",
          "items": { "$ref": "#/$defs/RawRecord" },
          "description": "Escape hatch — policy gated"
        }
      }
    }
  },
  "$defs": {
    "Route": {
      "type": "object",
      "required": ["host", "traffic"],
      "properties": {
        "host": { "type": "string", "format": "hostname" },
        "traffic": {
          "type": "object",
          "required": ["origin"],
          "properties": {
            "origin": { "type": "string", "format": "uri" },
            "paths": { "type": "array", "items": { "type": "string" } },
            "weights": { "type": "array", "items": { "$ref": "#/$defs/WeightedBackend" } }
          }
        },
        "tls": {
          "type": "object",
          "properties": {
            "mode": { "enum": ["auto", "custom", "passthrough"] }
          }
        }
      }
    },
    "EmailAuth": {
      "type": "object",
      "properties": {
        "spf": { "type": "object", "properties": { "includes": { "type": "array", "items": { "type": "string" } } } },
        "dkim": { "type": "object" },
        "dmarc": { "type": "object", "properties": { "policy": { "enum": ["none", "quarantine", "reject"] } } }
      }
    },
    "Verification": {
      "type": "object",
      "required": ["provider", "host"],
      "properties": {
        "provider": { "type": "string" },
        "host": { "type": "string", "format": "hostname" },
        "token_ref": { "type": "string" }
      }
    },
    "DomainTest": {
      "type": "object",
      "required": ["name", "assert"],
      "properties": {
        "name": { "type": "string" },
        "host": { "type": "string" },
        "assert": { "type": "object" }
      }
    },
    "RawRecord": {
      "type": "object",
      "required": ["type", "name", "content"],
      "properties": {
        "type": { "enum": ["A", "AAAA", "CNAME", "TXT", "MX", "NS", "CAA"] },
        "name": { "type": "string" },
        "content": { "type": "string" },
        "ttl": { "type": "integer", "minimum": 60 }
      }
    }
  }
}
```

---

## Intent IR (Compiler Internal)

Simplified excerpt:

```json
{
  "ir_version": "1",
  "fqdn_bindings": [
    {
      "fqdn": "api.example.com",
      "bindings": [
        { "kind": "http_route", "origin": "https://v2.internal", "paths": ["/*"] }
      ]
    }
  ]
}
```

Full schema: `intent-ir.schema.json`

---

## Intent Delta (API)

Partial document merged onto `base_version`:

```json
{
  "routes": [
    { "host": "api.example.com", "traffic": { "origin": "https://v2.internal" } }
  ]
}
```

Merge semantics: RFC 7386 JSON Merge Patch for top-level arrays by `host` key.