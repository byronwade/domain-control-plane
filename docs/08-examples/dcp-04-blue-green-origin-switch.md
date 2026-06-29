# Blue-Green Origin Switch Example

| Field | Value |
|-------|-------|
| Doc ID | `dcp-example-04` |
| Category | Examples |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-03, dcp-vision-04 |

---

## Summary

Demonstrates **instant routing change** with zero DNS operations — the core DCP physics advantage.

---

## Initial State

```yaml
spec:
  routes:
    - host: api.example.com
      traffic:
        origin: https://blue.k8s.internal  # v1.4.2
      tls:
        mode: auto
```

**DNS (unchanged throughout):**

```
api.example.com  CNAME  tenant-abc.edge.dcp.dev  TTL 3600
```

**Active route bundle:** `bundle_version: 41`

---

## Deploy Green (v1.5.0)

### Intent delta

```yaml
intent_delta:
  routes:
    - host: api.example.com
      traffic:
        origin: https://green.k8s.internal
```

### Submit

```bash
dcp transactions submit \
  --domain example.com \
  --delta '{"routes":[{"host":"api.example.com","traffic":{"origin":"https://green.k8s.internal"}}]}' \
  --idempotency-key deploy-v150 \
  --wait-for routing
```

### Result

```json
{
  "transaction_id": "txn_9c1",
  "phase": "committed",
  "plan_hash": "sha256:green_only",
  "status": {
    "routing": "active",
    "dns": "complete",
    "tls": "active"
  },
  "timing": {
    "routing_active_ms": 740,
    "dns_ops": 0
  },
  "bundle_version": 42
}
```

**New HTTP connections** route to green in < 1s. Existing keep-alive connections may persist on blue until idle (configurable drain timeout).

---

## Canary Variant

```yaml
intent_delta:
  routes:
    - host: api.example.com
      traffic:
        weights:
          - url: https://green.k8s.internal
            weight: 10
          - url: https://blue.k8s.internal
            weight: 90
```

Promote to 100% green:

```yaml
weights:
  - url: https://green.k8s.internal
    weight: 100
```

Each weight change = new bundle version; still zero DNS.

---

## Rollback

```bash
dcp transactions rollback --id txn_9c1 --wait-for routing
```

| Field | Value |
|-------|-------|
| `restores_version` | int_v41 |
| `bundle_version` | 41 |
| `routing_active_ms` | 620 |

---

## What Would Happen Without DCP

| Approach | Origin switch mechanism | Typical delay |
|----------|------------------------|---------------|
| Raw DNS A record | Change IP in zone | TTL-bound (300s–3600s) |
| Direct origin URL in DNS | CNAME swap | TTL + propagation |
| Load balancer manual | Console change | 5–15 min |
| **DCP route runtime** | Bundle push | **< 1s** |

---

## Observability

```bash
dcp probes run --domain example.com --targets api.example.com
```

```json
{
  "http": {
    "headers": { "X-DCP-Route-Version": "42" },
    "upstream_detected": "green.k8s.internal"
  }
}
```