# Security Controls

| Field | Value |
|-------|-------|
| Doc ID | `dcp-sec-03` |
| Category | Security |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-sec-01, dcp-sec-02 |

---

## Control Matrix

| ID | Control | Layer | Priority |
|----|---------|-------|----------|
| SC-01 | Capability token auth | API | P0 |
| SC-02 | Domain OAuth with PKCE | Auth | P0 |
| SC-03 | Transaction idempotency | Kernel | P0 |
| SC-04 | Lease exclusivity | Kernel | P0 |
| SC-05 | Policy default-deny | Compiler | P0 |
| SC-06 | Certificate firewall | TLS | P0 |
| SC-07 | Takeover immune system | Security | P0 |
| SC-08 | Recipe WASM sandbox | Recipe RT | P0 |
| SC-09 | Signed route bundles | Runtime | P0 |
| SC-10 | Provenance hash chain | Audit | P0 |
| SC-11 | AI proposal-only path | AI | P0 |
| SC-12 | Break-glass audit | Ops | P1 |
| SC-13 | mTLS runtime agents | Runtime | P1 |
| SC-14 | Rate limiting | API | P1 |
| SC-15 | Credential rotation | Vault | P1 |
| SC-16 | CT log monitoring | TLS | P1 |
| SC-17 | SIEM audit export | Observability | P2 |

---

## SC-11: AI Isolation (Detail)

Enforced at three layers:

1. **API gateway:** No endpoint maps AI service account to `POST /transactions` in production without `human_approved_plan_hash` JWT claim
2. **Kernel:** Rejects plans without `policy_decision_id` from deterministic policy engine
3. **Code review:** CI grep for forbidden import paths from `ai_planner` to `kernel_apply`

---

## SC-08: Recipe Sandbox (Detail)

| Restriction | Value |
|-------------|-------|
| Network egress | Allowlist per recipe manifest |
| Filesystem | None |
| Syscalls | WASI subset |
| Memory | 64 MB |
| Secrets | Injected pointer only |

---

## Compliance Mapping (Indicative)

| Framework | Relevant controls |
|-----------|-------------------|
| SOC 2 CC6 | SC-01, SC-02, SC-10, SC-12 |
| SOC 2 CC7 | SC-07, SC-16, SC-17 |
| ISO 27001 A.9 | SC-01, SC-02, SC-15 |
| PCI DSS 4.0 | SC-09, SC-13 (hybrid mode) |

---

## Incident Response

| Severity | Example | Response time |
|----------|---------|---------------|
| SEV1 | Cross-tenant data leak | 15 min |
| SEV2 | Mass cert firewall bypass | 30 min |
| SEV3 | Single domain takeover undetected | 4 hours |
| SEV4 | Recipe publisher key rotation | 24 hours |

Playbooks: revoke tokens → quarantine affected FQDNs → rollback transactions → notify customers.