# Provider Recipe Schema

| Field | Value |
|-------|-------|
| Doc ID | `dcp-schema-07` |
| Category | Schemas |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-schema-01, dcp-core-05 |

---

## ProviderRecipe Bundle

```json
{
  "$id": "https://schemas.dcp.dev/v1/provider-recipe.schema.json",
  "type": "object",
  "required": ["apiVersion", "kind", "metadata", "spec"],
  "properties": {
    "apiVersion": { "const": "dcp.dev/v1" },
    "kind": { "const": "ProviderRecipe" },
    "metadata": {
      "type": "object",
      "required": ["name", "version", "publisher", "signature"],
      "properties": {
        "name": { "type": "string" },
        "version": { "type": "string", "pattern": "^[0-9]+\\.[0-9]+\\.[0-9]+$" },
        "publisher": { "type": "string" },
        "signature": { "type": "string" },
        "min_kernel_version": { "type": "string" }
      }
    },
    "spec": {
      "type": "object",
      "required": ["capabilities", "operations"],
      "properties": {
        "capabilities": {
          "type": "array",
          "items": { "enum": ["dns.read", "dns.write", "route.write", "registrar.read", "registrar.write"] }
        },
        "credential_schema": { "$ref": "#/$defs/CredentialSchema" },
        "operations": {
          "type": "object",
          "additionalProperties": { "$ref": "#/$defs/OperationDef" }
        },
        "attestations": {
          "type": "object",
          "properties": {
            "outputs": { "type": "array", "items": { "type": "string" } }
          }
        },
        "network_allowlist": {
          "type": "array",
          "items": { "type": "string", "format": "hostname" }
        }
      }
    }
  },
  "$defs": {
    "CredentialSchema": {
      "type": "object",
      "properties": {
        "type": { "enum": ["api_token", "oauth_refresh", "iam_role", "api_key_pair"] },
        "scopes": { "type": "array", "items": { "type": "string" } },
        "rotation_days": { "type": "integer" }
      }
    },
    "OperationDef": {
      "type": "object",
      "required": ["wasm_module", "timeout_ms"],
      "properties": {
        "wasm_module": { "type": "string" },
        "timeout_ms": { "type": "integer", "maximum": 120000 },
        "idempotent": { "type": "boolean", "default": true },
        "retry": {
          "type": "object",
          "properties": {
            "max_attempts": { "type": "integer" },
            "backoff_ms": { "type": "integer" }
          }
        }
      }
    }
  }
}
```

---

## Recipe Invocation Payload

```json
{
  "recipe": "cloudflare@2.4.1",
  "operation": "dns.upsert",
  "input": {
    "zone_id": "abc",
    "type": "CNAME",
    "name": "api",
    "content": "tenant.edge.dcp.dev"
  },
  "credential_handle": "cred_ref_88",
  "fencing_token": 7712
}
```

---

## Recipe Output Attestation

```json
{
  "status": "success",
  "provider_request_id": "cf_991",
  "observed_state_hash": "sha256:...",
  "compensation_data": {},
  "wasm_exit_code": 0,
  "duration_ms": 342
}
```

---

## Publisher Tiers

| Tier | Signature key | Max capabilities |
|------|---------------|------------------|
| official | DCP HSM | all |
| verified-partner | vendor key | as declared |
| community | community CA | dns.read, dns.write only |
| customer | customer CA | self-hosted only |