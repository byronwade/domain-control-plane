# Design Principles

| Field | Value |
|-------|-------|
| Doc ID | `dcp-vision-03` |
| Category | Vision |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-vision-01, dcp-vision-02 |

---

## Summary

Ten principles governing every DCP design decision. When principles conflict, earlier principles win unless explicitly documented.

---

## P1: DNS Is Substrate, Not Product

DCP never claims to replace DNS. It compiles intent into DNS (and non-DNS) operations while exposing honest propagation semantics.

**Implication:** APIs return `propagation_status` alongside `routing_status`. Routing may be `active` while DNS is `propagating`.

---

## P2: Intent Over Records

Users think in outcomes: "route traffic," "verify domain," "enable email," "issue cert." Raw `TXT`/`CNAME` records are compiler output, not primary input.

**Implication:** Escape hatch `records.raw[]` exists but is policy-gated and audit-heavy.

---

## P3: Determinism Before Intelligence

AI may *propose* plans; only the deterministic compiler and policy engine may *approve* them. No LLM bypass of safety checks.

**Implication:** AI output is `PlanProposal`, not `Transaction`. See [dcp-core-10](../03-core-systems/dcp-10-ai-planner-debugger.md).

---

## P4: Everything Is a Transaction

No silent mutations. Every change has: idempotency key, plan, lease, apply log, verify results, and optional rollback pointer.

**Implication:** Even "read" diagnostics that mutate provider state (e.g., triggering verification) are transactions.

---

## P5: Least Privilege by Default

Credentials are scoped to capability tokens. Registrar root access is never required for subdomain operations.

**Implication:** Domain OAuth scopes are FQDN-prefix and operation-type bounded.

---

## P6: Stable DNS, Dynamic Routes

Minimize DNS churn. Point once; route often. TTL and record type choices favor stability.

**Implication:** Hosted runtime uses anycast/proxy IPs or long-lived CNAME targets.

---

## P7: Provenance Is Non-Optional

Every effective state links to: actor, parent intent version, transaction ID, and recipe attestations.

**Implication:** Cannot commit transaction without provenance record.

---

## P8: Fail Closed on Security

Certificate issuance, delegation, and dangling record detection default to deny. Explicit policy required to allow risk.

**Implication:** Certificate firewall blocks before CA contact. Takeover immune system quarantines suspicious records.

---

## P9: Provider Heterogeneity via Recipes

No monolithic provider SDK in the kernel. Signed, sandboxed recipes normalize provider quirks.

**Implication:** Community and vendor recipes; core ships reference implementations.

---

## P10: Reversibility Within Physics

Rollback restores DCP intent and compensates provider state where possible. Cannot unsend propagated DNS to every resolver instantly — rollback sets correct state and monitors convergence.

**Implication:** Rollback returns `routing_immediate: true`, `dns_propagation_eta: duration`.

---

## Anti-Patterns

| Anti-Pattern | Violation |
|--------------|-----------|
| "Sync DNS globally in 5 seconds" | P1 |
| Direct Terraform apply without transaction | P4 |
| Agent with registrar admin API key | P5 |
| Issue cert without provenance check | P7, P8 |
| Hardcode Cloudflare API in kernel | P9 |