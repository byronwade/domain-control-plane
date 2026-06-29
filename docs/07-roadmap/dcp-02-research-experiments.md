# Research Experiments

| Field | Value |
|-------|-------|
| Doc ID | `dcp-roadmap-02` |
| Category | Roadmap |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-vision-04, dcp-core-02 |

---

## Summary

Structured experiments to de-risk visionary claims. Each has hypothesis, method, success criteria, and honest failure interpretation.

---

## R1: Propagation Prediction Model

**Hypothesis:** Given TTL, provider, and record type, DCP can predict global DNS p50/p95 within ±20%.

| Attribute | Detail |
|-----------|--------|
| Method | Probe 200+ resolver locations; train on 10k record changes |
| Duration | 8 weeks |
| Success | Median absolute error < 15% on p50 |
| Failure interpretation | Show authoritative-only status; widen ETA confidence bands |

---

## R2: Stable DNS / Instant Route Split

**Hypothesis:** 95% of routing changes need zero DNS ops after initial onboarding.

| Attribute | Detail |
|-----------|--------|
| Method | Instrument compiler plans across 50 beta customers |
| Success | ≥ 95% route-only plans after day-7 |
| Failure interpretation | Invest in wildcard proxy patterns; accept DNS churn for apex |

---

## R3: WASM Recipe Sandboxing

**Hypothesis:** WASM+WASI isolates recipes with < 10% latency overhead vs native SDK.

| Attribute | Detail |
|-----------|--------|
| Method | Port 5 provider adapters; fuzz network egress |
| Success | Zero escapes; p99 op latency < 2s |
| Failure interpretation | gRPC out-of-process recipes with seccomp |

---

## R4: Takeover Reclaim Database Coverage

**Hypothesis:** Community-signed reclaim rules detect > 90% of real-world takeover CVE patterns.

| Attribute | Detail |
|-----------|--------|
| Method | Corpus of 500 historical takeover reports; weekly feed updates |
| Success | Recall ≥ 90%, false positive < 5% |
| Failure interpretation | Partner with CISA KEV; customer cloud inventory linking |

---

## R5: Intent IR Completeness

**Hypothesis:** IR with 12 binding kinds covers 99% of SaaS domain operations.

| Attribute | Detail |
|-----------|--------|
| Method | Survey 100 SaaS domain setup docs; map to IR |
| Success | ≥ 99% mappable without raw DNS escape |
| Failure interpretation | Expand IR; raw escape for long tail |

---

## R6: Saga Rollback Success Rate

**Hypothesis:** Compensating transactions succeed ≥ 99.5% for DNS+route changes across top 3 providers.

| Attribute | Detail |
|-----------|--------|
| Method | Chaos testing: inject 503/429 mid-apply |
| Success | ≥ 99.5% rollback to consistent state |
| Failure interpretation | Human escalation queue; freeze domain |

---

## R7: AI Planner Compile-Through Rate

**Hypothesis:** AI proposals compile without error ≥ 90% for top 50 intent templates.

| Attribute | Detail |
|-----------|--------|
| Method | Golden NL prompts → compile → compare to human intent |
| Success | ≥ 90% first-pass compile |
| Failure interpretation | Template library + retrieval; no autonomous apply |

---

## R8: Formal Policy Verification

**Hypothesis:** Selected policy rules can be proven to prevent fence escape via SMT solver.

| Attribute | Detail |
|-----------|--------|
| Method | Model capability grammar + fence rules in Z3 |
| Success | Proof for core 20 rules |
| Failure interpretation | Runtime tests only; narrow policy language |

---

## R9: Split-Horizon Intent

**Hypothesis:** Internal/external resolver views can be managed from single intent with overlays.

| Attribute | Detail |
|-----------|--------|
| Method | Dual probe network; split DNS provider adapters |
| Success | Consistent compile; no leakage in tests |
| Failure interpretation | Separate intents per horizon |

---

## R10: CT Log Preemption

**Hypothesis:** Cert firewall + immune system detects takeover risk before mis-issuance in > 95% of lab scenarios.

| Attribute | Detail |
|-----------|--------|
| Method | Red team dangling CNAME + ACME automation |
| Success | Block before cert active at edge |
| Failure interpretation | CT monitoring as backstop; shorter cert lifetimes |

---

## Experiment Prioritization

| Priority | Experiments | Phase |
|----------|-------------|-------|
| P0 | R2, R3, R6 | Pre-GA |
| P1 | R1, R4, R5 | Phase 2 |
| P2 | R7, R10 | Phase 3 |
| P3 | R8, R9 | Phase 4 |