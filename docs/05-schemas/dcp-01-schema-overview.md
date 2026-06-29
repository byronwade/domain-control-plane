# Schema Overview

| Field | Value |
|-------|-------|
| Doc ID | `dcp-schema-01` |
| Category | Schemas |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-api-01 |

---

## Summary

DCP schemas use **JSON Schema Draft 2020-12** with `apiVersion` / `kind` Kubernetes-style envelopes for documents, and plain JSON for API payloads.

---

## Schema Registry Layout

```
schemas/
  v1/
    domain-intent.schema.json
    intent-ir.schema.json
    operation-plan.schema.json
    transaction.schema.json
    plan-proposal.schema.json
    provenance-record.schema.json
    route-bundle.schema.json
    provider-recipe.schema.json
    capability-token.schema.json
    policy-pack.schema.json
```

Hosted registry: `https://schemas.dcp.dev/v1/{name}`

---

## Versioning Rules

| Change type | Version bump |
|-------------|--------------|
| Add optional field | Minor |
| Add required field | Major |
| Rename field | Major + migration |
| Semantic constraint | Minor + compiler pin |

`apiVersion: dcp.dev/v1` — major API version.

---

## Content Addressing

| Artifact | Hash input |
|----------|------------|
| Intent version | Canonical JSON of `spec` + `metadata.domain` |
| Operation plan | Ordered ops + compiler version |
| Route bundle | Routes + tls + version |
| Provenance | Full record minus signature |

Algorithm: `sha256` over UTF-8 canonical JSON (sorted keys).

---

## Validation Pipeline

```
Client payload → JSON Schema validate → Semantic validate (compiler) → Policy
```

SDKs embed schemas for offline validation.

---

## Cross-References

| Schema | Doc |
|--------|-----|
| DomainIntent | [dcp-02-intent-schema.md](./dcp-02-intent-schema.md) |
| Transaction | [dcp-03-transaction-schema.md](./dcp-03-transaction-schema.md) |
| Policy | [dcp-04-policy-schema.md](./dcp-04-policy-schema.md) |
| Provenance | [dcp-05-provenance-schema.md](./dcp-05-provenance-schema.md) |