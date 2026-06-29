# Edge Platform Origins Integration

| Field | Value |
|-------|-------|
| Doc ID | `dcp-integration-02` |
| Category | Integrations |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-03, dcp-vision-05 |

---

## Summary

DCP route runtime fronts **Vercel, Netlify, Fly, Railway, Cloudflare Pages** origins — giving instant failover and unified domain intent while keeping existing hosting.

---

## Pattern: DCP Proxy → Platform Origin

```mermaid
flowchart LR
    User[User] --> DNS[api.customer.com]
    DNS --> DCP[DCP Route Runtime]
    DCP --> Vercel[myapp.vercel.app]
```

**Customer DNS (once):**

```
api.customer.com  CNAME  tenant.edge.dcp.dev
```

**Intent:**

```yaml
routes:
  - host: api.customer.com
    traffic:
      origin: https://myapp.vercel.app
    tls:
      mode: auto
```

DCP terminates TLS at edge (customer branding) or passes through (passthrough mode).

---

## Platform-Specific Notes

### Vercel

| Item | Detail |
|------|--------|
| Origin URL | `https://{project}.vercel.app` |
| Preview deploys | Route `staging.api.customer.com` → `https://{project}-git-staging.vercel.app` |
| Verification | Add `api.customer.com` in Vercel project; DCP emits verification route |
| Instant switch | Change origin URL in intent — no Vercel domain re-attach |

### Netlify

| Item | Detail |
|------|--------|
| Origin URL | `https://{site}.netlify.app` |
| Branch deploys | Weighted routes to branch subdomain |
| SSL | DCP cert on customer domain; Netlify sees HTTP from DCP |

### Fly.io

| Item | Detail |
|------|--------|
| Origin URL | `https://{app}.fly.dev` |
| IPv6 | DCP edge handles dual-stack |
| Health | Fly machine health + DCP active probes |

### Cloudflare Pages

| Item | Detail |
|------|--------|
| Coexistence | DCP runtime OR CF Workers as runtime target |
| Recipe | `cloudflare-pages@1` for verification TXT |

---

## Failover Example

Primary Vercel, fallback Fly:

```yaml
routes:
  - host: api.customer.com
    traffic:
      weights:
        - url: https://myapp.vercel.app
          weight: 90
        - url: https://myapp.fly.dev
          weight: 10
health_checks:
  - backend_url: https://myapp.vercel.app
    path: /health
```

On Vercel unhealthy → runtime shifts weight to Fly without DNS change.

---

## vs Native Platform Custom Domains

| Concern | Platform-native | DCP fronted |
|---------|-----------------|-------------|
| Vendor lock | High | Origin swappable in intent |
| Instant rollback | Redeploy | Bundle version revert |
| Multi-platform | Per-platform UI | Single intent API |
| Email on same domain | Separate setup | Same intent document |
| Tenant custom domains | Manual per tenant | Delegation model |

---

## Verification Recipe Flow

```
1. Customer adds domain in Vercel dashboard (or API)
2. DCP verification facet triggers github/vercel recipe
3. Compiler emits required TXT
4. Transaction applies + verifies
5. Vercel marks domain verified
6. DCP route active
```