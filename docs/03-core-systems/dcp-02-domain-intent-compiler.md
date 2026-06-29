# Domain Intent Compiler

| Field | Value |
|-------|-------|
| Doc ID | `dcp-core-02` |
| Category | Core Systems |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-03, dcp-core-05 |

---

## Summary

The Domain Intent Compiler is a **deterministic multi-pass compiler** lowering developer intent to provider operations. It is the semantic bridge between "what I want" and "what the internet requires."

---

## Intent Document Model

```yaml
apiVersion: dcp.dev/v1
kind: DomainIntent
metadata:
  domain: example.com
  environment: production
  version: int_v13
spec:
  routes:
    - host: api.example.com
      traffic:
        origin: https://k8s-origin.internal
        paths: ["/*"]
      tls:
        mode: auto
  email:
    outbound:
      spf:
        includes: ["_spf.google.com"]
      dkim:
        selectors:
          - name: s1
            provider: google_workspace
  verifications:
    - provider: github_pages
      host: docs.example.com
  policy:
    takeover: strict
    cert_issuance: firewall
```

---

## Intent Facets

| Facet | IR Kind | Typical Lowered Ops |
|-------|---------|---------------------|
| Routes | `http_route`, `tcp_proxy` | CNAME/ALIAS, runtime config |
| TLS | `tls_auto`, `tls_custom` | CAA, ACME challenge, cert deploy |
| Email | `email_auth` | SPF, DKIM, DMARC TXT |
| Verification | `provider_verification` | Provider-specific TXT/CNAME |
| Registrar | `registrar_glue` | NS, DS, glue records |
| Raw (escape) | `dns_raw` | Direct record ops — policy gated |

---

## Compiler Internals

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Parser    │───►│  IR Builder │───►│   Lowerer   │
│  (YAML/JSON)│    │  + Resolver │    │  + Recipes  │
└─────────────┘    └─────────────┘    └─────────────┘
                          │                    │
                          ▼                    ▼
                   ┌─────────────┐    ┌─────────────┐
                   │  Conflict   │    │  Optimizer  │
                   │  Detector   │    │  + Planner  │
                   └─────────────┘    └─────────────┘
```

### Conflict Detection

| Conflict type | Resolution |
|---------------|------------|
| Overlapping paths | Error unless `precedence` set |
| Apex CNAME + A | Error (DNS illegality) |
| SPF >10 lookups | Warning + compress includes |
| CAA vs auto-TLS | Error or request CAA update op |

---

## Provider Snapshot

Before lowering, compiler fetches read-only snapshot:

```json
{
  "snapshot_id": "snap_20260628T1200Z",
  "providers": {
    "cloudflare": {
      "zones": { "example.com": { "records": [], "settings": {} } }
    },
    "namecheap": {
      "domains": { "example.com": { "nameservers": [] } }
    }
  }
}
```

Drift between snapshot and live state at apply time → `COMPILE_PROVIDER_DRIFT` or kernel retry.

---

## Multi-Provider Strategies

| Strategy | When |
|----------|------|
| Single authoritative DNS | Default |
| Split DNS | `internal` vs `external` intent overlays |
| Registrar + DNS separate | Glue NS ops sequenced first |
| Runtime-only change | Zero DNS ops in plan |

---

## Versioning & Branching

```
main (production) ── int_v13
staging branch    ── int_v13-stg (overlay)
```

Promotion = merge overlay → compile delta transaction.

---

## CLI & CI Integration

```bash
dcp compile --intent ./domain.intent.yaml --dry-run
dcp compile --diff int_v12 int_v13
dcp compile --output plan.json
```

CI gate: `plan_hash` must match approved hash in PR.

---

## Determinism Contract

```
hash(intent, compiler_version, recipe_set_hash, snapshot_id) → plan_hash
```

Reproducible builds enable:

- Simulation replay
- Audit ("why was this TXT record created?")
- AI plan validation (compile the proposal, compare hashes)

---

## Extension Points

| Extension | Mechanism |
|-----------|-----------|
| New provider | Signed recipe bundle |
| Custom policy rule | WASM policy plugin |
| New intent facet | IR schema version bump + lowerer plugin |