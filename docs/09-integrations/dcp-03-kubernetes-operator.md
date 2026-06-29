# Kubernetes Operator Integration

| Field | Value |
|-------|-------|
| Doc ID | `dcp-integration-03` |
| Category | Integrations |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-04, dcp-core-03 |

---

## Summary

The DCP Kubernetes operator syncs **Intent CRDs** to the control plane and optionally runs **self-hosted route runtime** as an in-cluster gateway.

---

## Custom Resources

| CRD | Purpose |
|-----|---------|
| `DomainIntent` | Declarative domain spec in cluster |
| `DomainRoute` | Single route (simplified) |
| `DcpRuntime` | Self-hosted runtime deployment |
| `DcpCredential` | Sealed credential reference |

---

## DomainIntent CRD

```yaml
apiVersion: dcp.dev/v1
kind: DomainIntent
metadata:
  name: example-com
  namespace: platform
spec:
  domain: example.com
  environment: production
  intent:
    routes:
      - host: api.example.com
        traffic:
          origin: https://api.svc.cluster.local
        tls:
          mode: auto
  syncPolicy:
    mode: transaction  # transaction | dry-run
    approvedPlanHash: ""  # set by CI for prod
status:
  intentVersion: int_v13
  transactionId: txn_7b2
  routing: active
  dns: propagating
  lastSync: "2026-06-28T12:00:00Z"
```

---

## Reconciliation Loop

```mermaid
flowchart LR
    CR[DomainIntent CR] --> Op[Operator]
    Op --> Compile[DCP Compile API]
    Compile --> Txn[Submit Transaction]
    Txn --> Status[Update CR Status]
```

1. Watch `DomainIntent` changes
2. Compile diff against `status.intentVersion`
3. If `syncPolicy.approvedPlanHash` set, verify match
4. Submit transaction with CR UID as idempotency key
5. Poll until `committed` or `failed`
6. Update `.status`

---

## Self-Hosted Runtime (Hybrid)

```yaml
apiVersion: dcp.dev/v1
kind: DcpRuntime
metadata:
  name: edge-gateway
  namespace: dcp-system
spec:
  replicas: 3
  serviceType: LoadBalancer
  registration:
    controlPlane: https://api.dcp.dev
    mTLSCertSecret: dcp-runtime-tls
  domains:
    - example.com
```

Operator deploys Envoy-based gateway; registers with control plane; pulls signed bundles.

**Customer DNS:**

```
api.example.com  A/AAAA  <LoadBalancer IP>
# OR
api.example.com  CNAME  edge.customer.com
```

---

## GitOps (ArgoCD / Flux)

```
Git repo (DomainIntent YAML)
  → Flux reconciles to cluster
  → DCP operator syncs to DCP API
  → Transaction applies
```

ArgoCD health check: `status.routing == active`.

---

## Secrets

```yaml
apiVersion: dcp.dev/v1
kind: DcpCredential
metadata:
  name: dcp-api-token
spec:
  vaultPath: secret/dcp/token
  capabilityPreset: deploy-bot
```

Never store raw registrar creds in CRs — use DCP Vault references.