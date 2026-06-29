# Threat Model

| Field | Value |
|-------|-------|
| Doc ID | `dcp-sec-01` |
| Category | Security |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-02, dcp-core-04 |

---

## Summary

STRIDE-based threat model for DCP hosted deployment. Self-hosted customers inherit substrate threats plus operational security.

---

## Assets

| Asset | Sensitivity |
|-------|-------------|
| Registrar/DNS credentials | Critical |
| Capability tokens | Critical |
| Intent versions | High |
| Provenance chain | High |
| TLS private keys | Critical |
| Recipe signing keys | Critical |
| Customer traffic metadata | Medium |
| AI proposal logs | Medium |

---

## Threat Actors

| Actor | Capability | Goal |
|-------|------------|------|
| External attacker | Network, DNS, CA | Subdomain takeover, cert mis-issue |
| Malicious tenant | Valid org account | Cross-tenant fence escape |
| Compromised CI token | Narrow capability | Deface route, exfil via DNS |
| Malicious third-party app | OAuth grant | Expand scope, persist access |
| Rogue recipe publisher | Signed recipe | Credential exfil, state corruption |
| Insider (DCP operator) | Infrastructure access | Mass customer impact |
| AI prompt injector | NL API access | Policy bypass (must fail) |

---

## STRIDE Analysis

### Spoofing

| Threat | Mitigation |
|--------|------------|
| Fake DCP API | TLS + cert pinning in SDKs |
| Forged capability JWT | Ed25519 sig, short TTL, introspection |
| Recipe impersonation | Cosign signature verification |

### Tampering

| Threat | Mitigation |
|--------|------------|
| Intent mutation in transit | TLS, request signing (enterprise) |
| Provenance chain edit | Hash chain + signed records |
| Runtime bundle MITM | Signed bundles, mTLS agent registration |

### Repudiation

| Threat | Mitigation |
|--------|------------|
| Deny making change | Provenance + audit export |
| Deny OAuth grant | Consent log with actor identity |

### Information Disclosure

| Threat | Mitigation |
|--------|------------|
| Cross-tenant intent leak | Org isolation, query fences |
| Credential in recipe logs | Redaction, vault injection only |
| AI training leakage | Tenant-scoped RAG, no cross-customer |

### Denial of Service

| Threat | Mitigation |
|--------|------------|
| Transaction flood | Rate limits, lease backpressure |
| Provider API exhaustion | Per-org quotas, circuit breakers |
| Probe amplification | Authenticated probe API only |

### Elevation of Privilege

| Threat | Mitigation |
|--------|------------|
| Scope escalation via OAuth | Immutable scopes on refresh shrink-only |
| Agent → production | Human approval gate, plan hash match |
| AI → kernel bypass | API enforcement, no code path |
| Fence escape | Compiler fence check, kernel lease bounds |

---

## Attack Scenarios

### Scenario 1: Subdomain Takeover

1. Customer deletes SaaS app but leaves CNAME
2. Attacker claims abandoned target
3. **Detection:** Immune system scores CNAME critical
4. **Response:** Quarantine + cert firewall deny + alert

### Scenario 2: Leaked CI Token

1. Token has `route:write:*.staging.example.com`
2. Attacker redirects staging to malicious origin
3. **Blast radius:** Staging only; production fenced
4. **Response:** Revoke token; rollback transaction via provenance

### Scenario 3: Malicious Recipe

1. Community recipe exfiltrates credential via DNS TXT
2. WASM sandbox blocks arbitrary network
3. **Detection:** Output attestation anomaly
4. **Response:** Publisher key revocation

---

## Residual Risk

| Risk | Acceptance |
|------|------------|
| Global DNS cache poisoning (external) | Out of scope; monitor via probes |
| Registrar compromise | Out of scope; detect via drift |
| Zero-day WASM escape | Defense in depth; recipe isolation updates |
| CT log permanent record | Short-lived certs; revocation procedures |