# Competitive Positioning

| Field | Value |
|-------|-------|
| Doc ID | `dcp-vision-05` |
| Category | Vision |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-vision-02 |

---

## Summary

DCP occupies a **new category** above managed DNS and beside edge platforms. It wins on transactional domain operations, permissions, and instant routing — not on raw resolution performance or CDN cache hit ratio.

---

## Landscape Matrix

| Capability | DCP | Cloudflare DNS | Route53 | Vercel Domains | Terraform | Resend Domains |
|------------|-----|----------------|---------|----------------|-----------|----------------|
| Intent-based API | ✅ | ❌ records | ❌ records | ❌ platform routes | ❌ HCL | ✅ email only |
| Transactional apply + rollback | ✅ | ❌ | ❌ | partial | plan/apply | ❌ |
| Instant route change (no DNS) | ✅ | Workers (CF-only) | ❌ | ✅ (Vercel-only) | ❌ | N/A |
| Domain OAuth / scoped tokens | ✅ | API token (broad) | IAM (complex) | team token | cloud creds | ❌ |
| Provenance graph | ✅ | audit logs | CloudTrail | deploy log | state file | ❌ |
| Certificate firewall | ✅ | partial | ❌ | implicit | ❌ | N/A |
| Takeover immune system | ✅ | partial | ❌ | ❌ | ❌ | N/A |
| Provider-neutral | ✅ | ❌ | ❌ | ❌ | ✅ | email only |
| Simulation / replay | ✅ | ❌ | ❌ | ❌ | plan | ❌ |
| AI planner (policy-bound) | ✅ | emerging | ❌ | ❌ | ❌ | ❌ |

---

## vs Cloudflare

**What Cloudflare is:** DNS + CDN + WAF + Workers as an integrated edge.

**Where DCP differs:**

| Dimension | Cloudflare | DCP |
|-----------|------------|-----|
| Primary abstraction | Zone records + Workers scripts | Versioned domain intent |
| Multi-provider | CF only | Recipe adapters (CF is one target) |
| Rollback | Manual or scripted | First-class compensating transaction |
| SaaS tenant isolation | Account/subaccount patterns | Provenance fences + capability scopes |
| Agent/CI permissions | Zone-wide API tokens | `route:write:fqdn:tenant-882.app.example.com` |

**Coexistence:** DCP compiles to Cloudflare via signed recipe. Customer keeps CF for CDN; DCP is control plane orchestration layer.

**When CF wins:** Single-provider shop wanting lowest latency anycast and mature WAF today.

**When DCP wins:** Multi-provider domain stack, SaaS custom domains, audit/compliance, safe CI automation.

---

## vs AWS Route53 + ACM + CloudFront

**What AWS is:** Authoritative DNS, cert manager, CDN — composed manually.

**Where DCP differs:**

| Dimension | AWS stack | DCP |
|-----------|-----------|-----|
| Integration burden | 3+ services, IAM policies | Single intent API |
| Change safety | CloudFormation drift | Kernel + provenance |
| Propagation visibility | Route53 health checks only | Global probe network + honest ETAs |
| Instant origin switch | CloudFront origin update (~minutes) | Route runtime (seconds) |

**Coexistence:** Route53 recipe for DNS; CloudFront or ALB as origin behind DCP runtime.

**When AWS wins:** All-in AWS, existing enterprise IAM, PrivateLink internal routing.

**When DCP wins:** Cross-cloud origins, developer experience, domain-level rollback story.

---

## vs Vercel / Netlify / Fly Custom Domains

**What they are:** Platform-native domain attach for their hosting.

**Where DCP differs:**

| Dimension | Edge platforms | DCP |
|-----------|----------------|-----|
| Scope | Their hosting only | Any origin (K8s, on-prem, multi-cloud) |
| Email / verification | Partial / manual | Intent facets + recipes |
| Versioning | Deploy-centric | Domain intent versions independent of app deploy |
| Self-hosted | ❌ | ✅ hybrid/self-hosted runtime |

**Coexistence:** Origin = `myapp.vercel.app`. DCP manages customer CNAME + instant failover to alternate origin.

**When Vercel wins:** Monolithic Vercel hosting; zero domain abstraction needed.

**When DCP wins:** SaaS giving customers `custom.customer.com`, multi-origin, domain CI gates.

---

## vs Terraform / Pulumi

**What IaC is:** Declarative infra with plan/apply and state files.

**Where DCP differs:**

| Dimension | IaC | DCP |
|-----------|-----|-----|
| Domain semantics | Raw `aws_route53_record` | Intent (`serve api.example.com`) |
| Runtime coupling | None | Route runtime for instant changes |
| Leases / idempotency | State lock | Per-resource domain leases |
| Live observability | Post-apply checks manual | Probes + propagation built-in |
| OAuth for apps | Cloud credentials | Domain capability tokens |

**Coexistence:** Terraform provider exports/imports intent; DCP executes. See [dcp-01-terraform-provider.md](../09-integrations/dcp-01-terraform-provider.md).

**When Terraform wins:** Full infra in one repo; existing TF ops maturity.

**When DCP wins:** Domain-specific safety, SaaS tenant model, runtime split, non-engineer consumers (support tools, agents).

---

## vs Resend (Domains)

**What Resend is:** Email API with domain setup for sending.

**Analogy, not competitor:** Resend owns email deliverability. DCP generalizes the *setup UX pattern* across DNS, TLS, routing, verification.

| Resend | DCP equivalent |
|--------|----------------|
| Add domain API | `domains.claim` |
| DNS record instructions | Compiler emits + applies |
| Verify + go live | Transaction verify phase |
| One API call | `transactions.submit` with intent delta |

DCP may integrate Resend as email provider recipe target.

---

## vs Certificate Authorities / ACME clients (certbot)

DCP certificate firewall **wraps** ACME — does not replace CA trust model. Value: provenance + takeover gating before issuance.

---

## Positioning Statement

> **For platform teams shipping custom domains**, DCP is the domain control plane that makes domain configuration programmable and reversible — unlike managed DNS dashboards or platform-locked domain UIs — by compiling intent into safe, scoped, observable transactions with instant routing behind stable DNS.

---

## Wedge → Expand

| Stage | Competitor displaced | Metric |
|-------|---------------------|--------|
| Wedge | Manual DNS + support tickets | < 60s custom domain live |
| Expand | Terraform domain modules | CI domain tests in 80% of repos |
| Platform | Cloudflare-only runbooks | 3+ provider recipes in production |
| Enterprise | Compliance spreadsheets | Provenance export satisfies SOC2 evidence |