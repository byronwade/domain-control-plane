# OpenAPI Reference

| Field | Value |
|-------|-------|
| Doc ID | `dcp-api-07` |
| Category | APIs |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-api-01 |

---

## Summary

Condensed OpenAPI 3.1 reference for DCP v1. Machine-readable spec ships at `https://api.dcp.dev/openapi/v1.json`.

---

## API Metadata

```yaml
openapi: 3.1.0
info:
  title: Domain Control Plane API
  version: 1.0.0-draft
  description: Intent-driven domain operations API
servers:
  - url: https://api.dcp.dev/v1
security:
  - bearerAuth: []
```

---

## Paths (Summary)

| Method | Path | Operation ID | Tag |
|--------|------|--------------|-----|
| GET | `/domains/{domain}/intent` | getIntent | Intent |
| GET | `/domains/{domain}/intent/versions` | listIntentVersions | Intent |
| GET | `/domains/{domain}/intent/versions/{version}` | getIntentVersion | Intent |
| GET | `/domains/{domain}/intent/diff` | diffIntent | Intent |
| POST | `/domains/{domain}/compile` | compileIntent | Intent |
| PUT | `/domains/{domain}/intent/branches/{branch}` | upsertIntentBranch | Intent |
| POST | `/domains/{domain}/intent/branches/{branch}/promote` | promoteIntentBranch | Intent |
| POST | `/domains/{domain}/transactions` | submitTransaction | Transactions |
| GET | `/transactions/{transaction_id}` | getTransaction | Transactions |
| GET | `/domains/{domain}/transactions` | listTransactions | Transactions |
| POST | `/transactions/{transaction_id}/rollback` | rollbackTransaction | Transactions |
| GET | `/transactions/{transaction_id}/events` | streamTransactionEvents | Transactions |
| POST | `/domains/{domain}/probes/run` | runProbes | Observability |
| GET | `/domains/{domain}/propagation` | getPropagation | Observability |
| GET | `/domains/{domain}/takeover-risks` | listTakeoverRisks | Security |
| POST | `/domains/{domain}/takeover-scan` | triggerTakeoverScan | Security |
| POST | `/policy/evaluate` | evaluatePolicy | Policy |
| GET | `/orgs/{org_id}/policy` | getOrgPolicy | Policy |
| GET | `/domains/{domain}/policy` | getDomainPolicy | Policy |
| GET | `/oauth/authorize` | oauthAuthorize | Auth |
| POST | `/oauth/token` | oauthToken | Auth |
| POST | `/oauth/introspect` | oauthIntrospect | Auth |
| POST | `/capabilities/revoke` | revokeCapability | Auth |
| POST | `/simulations` | createSimulation | Simulation |
| POST | `/ai/plan` | aiPlan | AI |
| POST | `/ai/debug` | aiDebug | AI |
| POST | `/orgs/{org_id}/webhooks` | createWebhook | Webhooks |

---

## Core Schemas

### SubmitTransactionRequest

```yaml
SubmitTransactionRequest:
  type: object
  required: [intent_delta]
  properties:
    intent_delta:
      type: object
      description: Partial intent merged onto base_version
    base_version:
      type: string
      pattern: '^int_v[0-9]+$'
    options:
      type: object
      properties:
        auto_rollback:
          type: boolean
          default: true
        dry_run:
          type: boolean
          default: false
        wait_for:
          type: array
          items:
            enum: [routing, authoritative_dns, propagation_p50, committed]
```

### Transaction

```yaml
Transaction:
  type: object
  required: [transaction_id, domain, phase]
  properties:
    transaction_id:
      type: string
    domain:
      type: string
      format: hostname
    phase:
      $ref: '#/components/schemas/TransactionPhase'
    plan_hash:
      type: string
    intent_version:
      type: string
    status:
      $ref: '#/components/schemas/FacetStatus'
    propagation:
      $ref: '#/components/schemas/PropagationStatus'
```

### TransactionPhase

```yaml
TransactionPhase:
  type: string
  enum:
    - pending
    - planning
    - planned
    - leasing
    - leased
    - applying
    - verifying
    - committed
    - rolling_back
    - rolled_back
    - failed
```

### FacetStatus

```yaml
FacetStatus:
  type: object
  properties:
    routing:
      enum: [pending, active, failed, quarantined]
    dns:
      enum: [pending, authoritative, propagating, complete, failed]
    tls:
      enum: [pending, active, failed, blocked]
    email:
      enum: [pending, active, failed]
```

### Error

```yaml
Error:
  type: object
  required: [code, message]
  properties:
    code:
      type: string
    message:
      type: string
    details:
      type: array
      items:
        type: object
        properties:
          field: { type: string }
          reason: { type: string }
    doc_url:
      type: string
      format: uri
    request_id:
      type: string
```

---

## Security Schemes

```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: Domain capability token
    apiKeyAuth:
      type: apiKey
      in: header
      name: Authorization
      description: 'Format: Dcp-Api-Key {key}'
```

---

## Standard Parameters

```yaml
parameters:
  DomainPath:
    name: domain
    in: path
    required: true
    schema:
      type: string
      format: hostname
  IdempotencyKey:
    name: Idempotency-Key
    in: header
    required: true
    schema:
      type: string
      maxLength: 128
  OrgId:
    name: Dcp-Org-Id
    in: header
    required: true
    schema:
      type: string
```

---

## Example: submitTransaction

```yaml
/domains/{domain}/transactions:
  post:
    operationId: submitTransaction
    tags: [Transactions]
    parameters:
      - $ref: '#/components/parameters/DomainPath'
      - $ref: '#/components/parameters/IdempotencyKey'
      - $ref: '#/components/parameters/OrgId'
    requestBody:
      required: true
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/SubmitTransactionRequest'
          example:
            intent_delta:
              routes:
                - host: api.example.com
                  traffic:
                    origin: https://v2.internal
            base_version: int_v12
            options:
              wait_for: [routing]
    responses:
      '201':
        description: Transaction created
        content:
          application/json:
            schema:
              type: object
              properties:
                data:
                  $ref: '#/components/schemas/Transaction'
      '409':
        description: Idempotency conflict or lease denied
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Error'
      '422':
        description: Policy rejected
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Error'
```

---

## SDK Generation

```bash
openapi-generator generate -i https://api.dcp.dev/openapi/v1.json -g typescript-fetch -o sdk/ts
```

Pinned spec hash recorded in SDK releases for reproducibility.