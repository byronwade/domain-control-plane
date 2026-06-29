# MVP Roadmap

| Field | Value |
|-------|-------|
| Doc ID | `dcp-roadmap-01` |
| Category | Roadmap |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-01, dcp-core-01 |

---

## Summary

Phased delivery from **routing wedge** to **full control plane**. Each phase has exit criteria before the next begins.

---

## Phase 0: Foundation (Weeks 1–8)

**Goal:** Internal dogfood — one domain, one provider, hosted runtime.

| Deliverable | Exit criteria |
|-------------|---------------|
| Intent schema v1 | Validates routes + TLS auto |
| Compiler v0 | Lowers to Cloudflare recipe only |
| Kernel v0 | plan → apply → verify → commit |
| Hosted route runtime | Anycast edge, < 5s route switch |
| Transaction API | Idempotency + SSE events |
| Basic probes | DNS authoritative + HTTP |

**Explicitly deferred:** OAuth, immune system, AI, self-hosted.

---

## Phase 1: MVP — "Resend for Domains" (Weeks 9–16)

**Goal:** External beta — SaaS custom domains in < 60 seconds.

| Deliverable | Exit criteria |
|-------------|---------------|
| Domain claim via DNS TXT | Claim → active in UI |
| 3 DNS providers | Cloudflare, Route53, Google Cloud DNS |
| 2 registrars (read-only) | Namecheap, GoDaddy linking |
| Domain OAuth + capabilities | CI deploy bot preset works |
| Route runtime GA | 99.9% availability single region |
| Rollback | 99% success on route-only txns |
| Dashboard | Intent diff, transaction timeline |
| TypeScript SDK | npm publish |

### MVP User Story

```typescript
import { Dcp } from '@dcp/sdk';

const dcp = new Dcp({ token: process.env.DCP_TOKEN });

await dcp.domains.claim('example.com');

const txn = await dcp.transactions.submit({
  domain: 'example.com',
  intentDelta: {
    routes: [{ host: 'api.example.com', traffic: { origin: 'https://myapp.vercel.app' } }],
  },
  waitFor: ['routing'],
});

// txn.status.routing === 'active' in < 60s
// txn.status.dns may === 'propagating'
```

### MVP Non-Goals

- Self-hosted control plane
- Email/DMARC automation
- AI planner
- Multi-region active-active

---

## Phase 2: Safety & Governance (Weeks 17–28)

| Deliverable | Exit criteria |
|-------------|---------------|
| Certificate firewall | 100% ACME gated |
| Takeover immune system | Detect top 20 dangling patterns |
| Provenance graph UI | Full lineage per FQDN |
| Domain tests in CI | `dcp test` blocks merge |
| Simulation API | p50 propagation estimate ± 20% |
| Policy packs | SaaS multi-tenant pack shipped |
| Hybrid runtime | Self-hosted agent beta |

---

## Phase 3: Platform (Weeks 29–44)

| Deliverable | Exit criteria |
|-------------|---------------|
| Recipe marketplace | 10 verified recipes |
| Email intent facet | SPF/DKIM/DMARC compile |
| Provider verification recipes | GitHub, Google, Microsoft |
| Replay & shadow environments | Recipe regression suite |
| AI debugger (read-only) | Debug reports for failed txns |
| Self-hosted Helm chart | GA for enterprise |
| Multi-region control plane | Active-active API |

---

## Phase 4: Ecosystem (Weeks 45+)

| Deliverable | Exit criteria |
|-------------|---------------|
| AI planner with human-in-loop | Production approval flow |
| Terraform provider | State sync with intent |
| Federation | Cross-org delegation for agencies |
| Air-gapped appliance | Gov pilot |
| Formal policy verification | Research → product |

---

## Team Sizing (Indicative)

| Phase | Engineers | Roles |
|-------|-----------|-------|
| 0–1 | 4–6 | Kernel, compiler, runtime, API |
| 2 | +3 | Security, observability, SDK |
| 3 | +4 | Recipes, enterprise, AI |
| 4 | +2 | Ecosystem, compliance |

---

## Risk Register

| Risk | Mitigation |
|------|------------|
| Provider API instability | Recipe circuit breakers; 3 providers |
| DNS propagation disappointment | Honest UX; routing-first wedge |
| Security incident early | Cert firewall + immune in Phase 2 before scale |
| Recipe maintenance burden | Official recipes only in MVP |