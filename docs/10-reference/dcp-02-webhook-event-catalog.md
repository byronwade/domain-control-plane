# Webhook Event Catalog

| Field | Value |
|-------|-------|
| Doc ID | `dcp-ref-02` |
| Category | Reference |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-06 |

---

## Summary

Complete payload schemas for DCP webhook events (CloudEvents `data` field).

---

## `dcp.transaction.committed`

```json
{
  "org_id": "org_abc",
  "domain": "example.com",
  "transaction_id": "txn_7b2",
  "intent_version": "int_v13",
  "plan_hash": "sha256:8f3a...",
  "provenance_id": "prov_f44",
  "actor": { "type": "application", "id": "ci_bot" },
  "status": {
    "routing": "active",
    "dns": "propagating",
    "tls": "active",
    "email": "pending"
  },
  "timing": {
    "routing_active_ms": 890,
    "total_ms": 4500
  }
}
```

---

## `dcp.transaction.failed`

```json
{
  "org_id": "org_abc",
  "domain": "example.com",
  "transaction_id": "txn_7b2",
  "phase": "failed",
  "error": {
    "code": "TXN_VERIFY_FAILED",
    "message": "HTTP probe returned 502"
  },
  "auto_rollback_status": "rolled_back"
}
```

---

## `dcp.transaction.phase_changed`

```json
{
  "transaction_id": "txn_7b2",
  "domain": "example.com",
  "previous_phase": "applying",
  "phase": "verifying",
  "at": "2026-06-28T12:00:02Z"
}
```

---

## `dcp.propagation.threshold_crossed`

```json
{
  "domain": "example.com",
  "fqdn": "api.example.com",
  "record_type": "CNAME",
  "threshold": "p50",
  "status": "complete",
  "resolver_sample_size": 240,
  "elapsed_minutes": 8
}
```

---

## `dcp.takeover.risk_detected`

```json
{
  "domain": "example.com",
  "fqdn": "legacy.example.com",
  "risk_score": 92,
  "severity": "critical",
  "signals": [
    {
      "code": "DANGLING_CNAME",
      "target": "deleted-app.herokuapp.com",
      "reclaimable": true
    }
  ],
  "recommended_action": "quarantine_and_notify"
}
```

---

## `dcp.takeover.quarantine_applied`

```json
{
  "domain": "example.com",
  "fqdn": "legacy.example.com",
  "quarantine_id": "q_09",
  "risk_score": 92,
  "routing_status": "quarantined"
}
```

---

## `dcp.cert_firewall.denied`

```json
{
  "domain": "example.com",
  "fqdn": "api.example.com",
  "decision_id": "cfw_881",
  "reason_code": "TAKEOVER_RISK_CRITICAL",
  "risk_score": 91,
  "requested_sans": ["api.example.com"]
}
```

---

## `dcp.route.bundle_activated`

```json
{
  "domain": "api.example.com",
  "bundle_id": "rtb_44c",
  "bundle_version": 42,
  "transaction_id": "txn_7b2",
  "previous_bundle_version": 41,
  "activated_at": "2026-06-28T12:00:01Z"
}
```

---

## `dcp.drift.detected`

```json
{
  "domain": "example.com",
  "drift_id": "drift_09",
  "resource": "rrset:example.com:TXT",
  "expected_hash": "sha256:a",
  "observed_hash": "sha256:b",
  "detected_by": "immune_system",
  "suggested_reconcile": true
}
```

---

## `dcp.compile.plan_rejected`

```json
{
  "domain": "example.com",
  "plan_hash": "sha256:...",
  "decision": "deny",
  "decision_id": "pol_991",
  "rules_failed": ["POLICY_RAW_DNS_DENIED"]
}
```

---

## Subscription Recommendations

| Use case | Minimum events |
|----------|----------------|
| Deploy bot | `committed`, `failed` |
| Security SOC | `takeover.*`, `cert_firewall.denied`, `drift.detected` |
| SRE dashboard | `phase_changed`, `propagation.*`, `route.bundle_activated` |
| Billing | `committed`, `route.bundle_activated` |