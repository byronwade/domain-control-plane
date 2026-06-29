# Multi-Tenant SaaS Intent Example

| Field | Value |
|-------|-------|
| Doc ID | `dcp-example-01` |
| Category | Examples |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-05, dcp-schema-02 |

---

## Summary

Reference intent documents for **Acme Platform** — a B2B SaaS offering `*.customers.acme.com` custom subdomains to tenants.

---

## Platform Base Intent

```yaml
apiVersion: dcp.dev/v1
kind: DomainIntent
metadata:
  domain: acme.com
  environment: production
  version: int_v40
  labels:
    org: org_acme_platform
    tier: platform
spec:
  routes:
    # Wildcard catch-all — stable DNS, per-tenant overlays
    - host: "*.customers.acme.com"
      traffic:
        origin: https://platform-router.acme.internal
        paths: ["/*"]
      tls:
        mode: auto

    # Platform marketing site (separate origin)
    - host: www.acme.com
      traffic:
        origin: https://marketing.vercel.app
      tls:
        mode: auto

  email:
    outbound:
      spf:
        includes: ["_spf.google.com"]
      dmarc:
        policy: quarantine
        rua: mailto:dmarc@acme.com

  policy:
    takeover: strict
    cert_issuance: firewall

  tests:
    - name: wildcard-tls-valid
      host: demo.customers.acme.com
      assert:
        tls:
          san_contains: demo.customers.acme.com
        http:
          url: https://demo.customers.acme.com/health
          status: 200
```

**DNS setup (once):**

```
customers.acme.com     NS    ns1.dcp-dns.dev
*.customers.acme.com   (managed by DCP adapter — wildcard)
```

After this, tenant onboarding requires **zero DNS operations**.

---

## Tenant Overlay — Acme Corp Customer

```yaml
apiVersion: dcp.dev/v1
kind: DomainIntentOverlay
metadata:
  domain: acme.com
  environment: production
  tenant_org: org_tenant_acmecorp
  base_version: int_v40
spec:
  routes:
    - host: acmecorp.customers.acme.com
      traffic:
        origin: https://tenant-acmecorp.k8s.acme.internal
        paths: ["/*"]
      tls:
        mode: auto
```

### Submit via API

```bash
curl -X POST https://api.dcp.dev/v1/domains/acme.com/transactions \
  -H "Authorization: Bearer ${TENANT_TOKEN}" \
  -H "Dcp-Org-Id: org_tenant_acmecorp" \
  -H "Idempotency-Key: onboard-acmecorp-20260628" \
  -d '{
    "intent_delta": {
      "routes": [{
        "host": "acmecorp.customers.acme.com",
        "traffic": { "origin": "https://tenant-acmecorp.k8s.acme.internal" }
      }]
    },
    "base_version": "int_v40",
    "options": { "wait_for": ["routing"] }
  }'
```

**Expected response timing:**

| Status | Time |
|--------|------|
| `routing: active` | < 10s |
| `dns: complete` | N/A (wildcard covers) |
| `tls: active` | < 60s (wildcard cert or per-host SAN) |

---

## Bring Your Own Domain — `app.acmecorp.com`

```yaml
spec:
  routes:
    - host: app.acmecorp.com
      traffic:
        origin: https://tenant-acmecorp.k8s.acme.internal
      tls:
        mode: auto
  verifications:
    - provider: dcp_claim
      host: app.acmecorp.com
```

**Customer DNS (their zone):**

```
app.acmecorp.com  CNAME  acmecorp.edge.dcp.dev
```

**Claim verification:**

```
_dcp-challenge.app.acmecorp.com  TXT  dcp_verify_8f3a2b1c
```

---

## TypeScript SDK Flow

```typescript
import { Dcp } from '@dcp/sdk';

const platform = new Dcp({ token: process.env.DCP_PLATFORM_TOKEN });
const tenant = new Dcp({
  token: process.env.DCP_TENANT_TOKEN,
  orgId: 'org_tenant_acmecorp',
});

// Platform: delegate (one-time)
await platform.delegations.create({
  toOrg: 'org_tenant_acmecorp',
  resource: 'fqdn:acmecorp.customers.acme.com',
  allowedFacets: ['route', 'tls'],
});

// Tenant: go live
const txn = await tenant.transactions.submit({
  domain: 'acme.com',
  intentDelta: {
    routes: [{
      host: 'acmecorp.customers.acme.com',
      traffic: { origin: process.env.TENANT_ORIGIN_URL },
    }],
  },
  waitFor: ['routing'],
});

console.log(txn.status.routing); // 'active'
```

---

## Policy Pack Applied

```yaml
# saas-multi-tenant pack
production_gates:
  require_ci_plan_hash: true
takeover_policy:
  quarantine_threshold: 80
  auto_remove_dangling_cname: true
capability_constraints:
  - actor_type: application
    deny_actions: ["registrar:*", "raw_dns:write"]
```