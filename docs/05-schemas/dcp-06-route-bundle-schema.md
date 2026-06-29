# Route Bundle Schema

| Field | Value |
|-------|-------|
| Doc ID | `dcp-schema-06` |
| Category | Schemas |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-schema-01, dcp-core-03 |

---

## RouteConfigBundle

Signed artifact pushed from kernel to route runtime.

```json
{
  "$id": "https://schemas.dcp.dev/v1/route-bundle.schema.json",
  "type": "object",
  "required": ["bundle_id", "bundle_version", "domain", "routes", "signature"],
  "properties": {
    "bundle_id": { "type": "string", "pattern": "^rtb_" },
    "bundle_version": { "type": "integer", "minimum": 1 },
    "domain": { "type": "string", "format": "hostname" },
    "effective_at": { "type": "string", "format": "date-time" },
    "transaction_id": { "type": "string" },
    "intent_version": { "type": "string" },
    "routes": {
      "type": "array",
      "items": { "$ref": "#/$defs/RouteRule" }
    },
    "tls": { "$ref": "#/$defs/TlsConfig" },
    "headers": { "$ref": "#/$defs/HeaderPolicy" },
    "health_checks": {
      "type": "array",
      "items": { "$ref": "#/$defs/HealthCheck" }
    },
    "signature": {
      "type": "string",
      "description": "Ed25519 over canonical bundle sans signature"
    }
  },
  "$defs": {
    "RouteRule": {
      "type": "object",
      "required": ["match", "backend"],
      "properties": {
        "match": {
          "type": "object",
          "properties": {
            "path_prefix": { "type": "string" },
            "path_regex": { "type": "string" },
            "headers": { "type": "object" }
          }
        },
        "backend": {
          "oneOf": [
            { "$ref": "#/$defs/SingleBackend" },
            { "$ref": "#/$defs/WeightedBackends" }
          ]
        },
        "redirect": {
          "type": "object",
          "properties": {
            "status": { "enum": [301, 302, 307, 308] },
            "location": { "type": "string" }
          }
        }
      }
    },
    "SingleBackend": {
      "type": "object",
      "required": ["url"],
      "properties": {
        "url": { "type": "string", "format": "uri" },
        "weight": { "const": 100 }
      }
    },
    "WeightedBackends": {
      "type": "object",
      "required": ["backends"],
      "properties": {
        "backends": {
          "type": "array",
          "items": {
            "type": "object",
            "required": ["url", "weight"],
            "properties": {
              "url": { "type": "string", "format": "uri" },
              "weight": { "type": "integer", "minimum": 0, "maximum": 100 }
            }
          }
        }
      }
    },
    "TlsConfig": {
      "type": "object",
      "properties": {
        "mode": { "enum": ["terminate", "passthrough"] },
        "cert_id": { "type": "string" },
        "min_version": { "enum": ["1.2", "1.3"] },
        "hsts": {
          "type": "object",
          "properties": {
            "max_age": { "type": "integer" },
            "include_subdomains": { "type": "boolean" }
          }
        }
      }
    },
    "HealthCheck": {
      "type": "object",
      "properties": {
        "backend_url": { "type": "string" },
        "path": { "type": "string", "default": "/health" },
        "interval_ms": { "type": "integer", "default": 5000 }
      }
    }
  }
}
```

---

## Signing

```
canonical = JSON.stringify(bundle, sorted_keys, exclude=signature)
signature = Ed25519.sign(kernel_signing_key, sha256(canonical))
```

Runtime verifies with pinned kernel public key; rejects downgrade attacks (bundle_version must be ≥ active).

---

## Activation Semantics

| Rule | Detail |
|------|--------|
| Monotonic version | `bundle_version` strictly increasing per FQDN |
| Zero-downtime | New bundle loaded; old drained after connection idle |
| Rollback | Reactivate `bundle_version - 1` pointer |
| Invalid sig | Reject; keep current bundle |