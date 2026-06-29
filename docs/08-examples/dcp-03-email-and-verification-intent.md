# Email and Verification Intent Example

| Field | Value |
|-------|-------|
| Doc ID | `dcp-example-03` |
| Category | Examples |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-02, dcp-schema-02 |

---

## Summary

Full intent for production domain with Google Workspace email, GitHub Pages docs, and API routing — demonstrating multi-facet compilation in one transaction.

---

## Complete Intent

```yaml
apiVersion: dcp.dev/v1
kind: DomainIntent
metadata:
  domain: example.com
  environment: production
  version: int_v20
spec:
  routes:
    - host: api.example.com
      traffic:
        origin: https://api.k8s.example.internal
      tls:
        mode: auto

    - host: docs.example.com
      traffic:
        origin: https://acme.github.io
      tls:
        mode: auto

  email:
    outbound:
      spf:
        includes:
          - "_spf.google.com"
          - "include:sendgrid.net"
        max_lookups_policy: compress
      dkim:
        selectors:
          - name: google
            provider: google_workspace
          - name: s1
            provider: sendgrid
            token_ref: secret:sendgrid_dkim
      dmarc:
        policy: quarantine
        pct: 100
        rua: mailto:dmarc-reports@example.com
        ruf: mailto:dmarc-forensic@example.com
      rotation_overlap_period: 72h

  verifications:
    - provider: google_workspace
      host: example.com
      token_ref: secret:google_site_verify

    - provider: github_pages
      host: docs.example.com
      token_ref: secret:github_pages_verify

  tls:
    caa:
      - "0 issue \"letsencrypt.org\""
      - "0 issuewild \";\""

  policy:
    takeover: strict
    cert_issuance: firewall

  tests:
    - name: spf-valid
      host: example.com
      assert:
        email:
          spf_includes: ["_spf.google.com"]
          spf_lookup_count_lte: 10

    - name: dmarc-present
      host: example.com
      assert:
        email:
          dmarc_policy: quarantine

    - name: docs-github-verify
      host: docs.example.com
      assert:
        dns:
          txt_contains: "github-pages"
```

---

## Compiled Operation Plan (Illustrative)

| Op | Provider | Action | Purpose |
|----|----------|--------|---------|
| 1 | cloudflare | `dns.upsert` | CNAME `api` → edge |
| 2 | cloudflare | `dns.upsert` | CNAME `docs` → edge |
| 3 | cloudflare | `dns.upsert` | TXT SPF at apex |
| 4 | cloudflare | `dns.upsert` | TXT DKIM `google._domainkey` |
| 5 | cloudflare | `dns.upsert` | TXT DKIM `s1._domainkey` |
| 6 | cloudflare | `dns.upsert` | TXT `_dmarc` |
| 7 | cloudflare | `dns.upsert` | TXT Google verify |
| 8 | cloudflare | `dns.upsert` | TXT/CNAME GitHub verify |
| 9 | cloudflare | `dns.upsert` | CAA records |
| 10 | dcp-runtime | `route.push` | api + docs bundles |
| 11 | dcp-acme | `tls.issue` | api.example.com SAN |
| 12 | dcp-acme | `tls.issue` | docs.example.com SAN |

**Routing ops (10)** execute first → instant for api/docs paths behind stable CNAME.

**DNS ops (1–9)** propagate on their own schedule.

---

## Transaction Status Timeline

| T+ | routing | dns | tls | email |
|----|---------|-----|-----|-------|
| 0s | pending | pending | pending | pending |
| 2s | active | authoritative | pending | authoritative |
| 5s | active | propagating | active | propagating |
| 15m | active | complete (p50) | active | complete |

---

## DKIM Rotation Example

When rotating SendGrid selector `s1` → `s2`:

```yaml
intent_delta:
  email:
    outbound:
      dkim:
        selectors:
          - name: s1
            provider: sendgrid
            state: retiring
          - name: s2
            provider: sendgrid
            state: active
```

Compiler maintains **both** TXT records for `rotation_overlap_period: 72h`, then removes `s1` in follow-up transaction.