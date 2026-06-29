# Policy API

| Field | Value |
|-------|-------|
| Doc ID | `dcp-api-04` |
| Category | APIs |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-07, dcp-core-08, dcp-api-01 |

---

## Summary

Policy API manages organization and domain-level rules evaluated during compilation and certificate firewall.

---

## Get Effective Policy

```
GET /v1/orgs/{org_id}/policy
GET /v1/domains/{domain}/policy
```

Domain policy overlays org policy.

```json
{
  "data": {
    "cert_policy": {
      "mode": "strict",
      "allow_wildcard": false
    },
    "takeover_policy": {
      "auto_remove_dangling_cname": true,
      "quarantine_threshold": 86
    },
    "raw_dns_policy": {
      "allow": false
    },
    "production_gates": {
      "require_ci_plan_hash": true,
      "require_human_approval_for_agents": true
    }
  }
}
```

---

## Evaluate Plan (Pre-flight)

```
POST /v1/policy/evaluate
```

```json
{
  "domain": "example.com",
  "plan_hash": "sha256:8f3a...",
  "actor": { "type": "application", "id": "app_ci" }
}
```

### Response

```json
{
  "data": {
    "decision": "allow",
    "decision_id": "pol_991",
    "rules_matched": ["route:staging-only-for-app_ci"],
    "warnings": []
  }
}
```

Decisions `deny` return `422` with `decision_id` for audit.

---

## Policy Packs (Templates)

```
GET /v1/policy-packs
```

| Pack | Description |
|------|-------------|
| `saas-multi-tenant` | Strict takeover, tenant fences |
| `enterprise-governance` | Raw DNS deny, break-glass audit |
| `startup-permissive` | Staging gates only |

```
POST /v1/orgs/{org_id}/policy/apply-pack
{ "pack_id": "saas-multi-tenant" }
```