# Terraform Provider Integration

| Field | Value |
|-------|-------|
| Doc ID | `dcp-integration-01` |
| Category | Integrations |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-02, dcp-api-03 |

---

## Summary

The Terraform provider manages **DCP intent resources**, not raw DNS records. Terraform plan = compile preview; apply = transaction submit.

---

## Design Principle

```
Terraform desired state  ↔  DCP Intent Document
Terraform plan           ↔  dcp compile (dry_run)
Terraform apply          ↔  dcp transactions.submit
Terraform state          ↔  intent version + transaction IDs
```

DCP remains source of truth for domain semantics. Terraform is one authoring surface.

---

## Resources

| Resource | Purpose |
|----------|---------|
| `dcp_domain` | Claim/link domain |
| `dcp_intent` | Full or partial intent spec |
| `dcp_route` | Convenience wrapper for single route |
| `dcp_delegation` | Multi-tenant delegate |
| `dcp_domain_test` | Assertion definitions |
| `dcp_webhook` | Webhook endpoint |

---

## Example Configuration

```hcl
terraform {
  required_providers {
    dcp = {
      source  = "dcp-dev/dcp"
      version = "~> 0.1"
    }
  }
}

provider "dcp" {
  api_token = var.dcp_token
  org_id    = var.dcp_org_id
}

resource "dcp_domain" "example" {
  domain = "example.com"
  claim_method = "dns_txt"
}

resource "dcp_route" "api" {
  domain = dcp_domain.example.domain
  host   = "api.example.com"
  origin = "https://api.k8s.internal"
  tls_mode = "auto"

  depends_on = [dcp_domain.example]
}

resource "dcp_intent" "email" {
  domain = dcp_domain.example.domain
  spec = yamlencode({
    email = {
      outbound = {
        spf = { includes = ["_spf.google.com"] }
        dmarc = { policy = "quarantine" }
      }
    }
  })
}
```

---

## Plan Phase

```
# dcp_route.api will create transaction
  + resource "dcp_route" "api"
      host:   "api.example.com"
      origin: "https://api.k8s.internal"

Plan: plan_hash = sha256:8f3a...
      dns_ops: 1
      routing_ops: 1
      predicted routing_active: <5s
```

---

## Apply Phase

```
dcp_route.api: Creating...
dcp_route.api: Transaction txn_7b2 committed
dcp_route.api: routing=active (890ms), dns=propagating
dcp_route.api: Creation complete after 5s
```

State stores:

```hcl
# terraform.tfstate (excerpt)
{
  "transaction_id": "txn_7b2",
  "intent_version": "int_v13",
  "plan_hash": "sha256:8f3a..."
}
```

---

## Drift Detection

Terraform refresh calls:

```
GET /intent/diff?from=state.intent_version&to=remote
```

If drift detected:

```
# dcp_route.api origin must be replaced
  ~ origin: "https://old.internal" -> "https://new.internal"
```

Immune system drift and Terraform drift are complementary — both surface in plan.

---

## Import

```bash
terraform import dcp_domain.example example.com
terraform import dcp_route.api example.com/api.example.com
```

Imports read current intent; does not mutate.

---

## CI Recommendation

Use `terraform plan` with `-detailed-exitcode` plus `dcp simulate` in pipeline. Require matching `plan_hash` for production apply.