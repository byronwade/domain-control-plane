# System Overview

| Field | Value |
|-------|-------|
| Doc ID | `dcp-arch-01` |
| Category | Architecture |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-vision-01, dcp-vision-03 |

---

## Summary

DCP is a **control plane** that accepts versioned domain intent, compiles it to provider operations, executes them transactionally, and maintains instant routing via an optional **route runtime**. DNS providers and registrars are downstream adapters.

---

## High-Level Architecture

```mermaid
flowchart TB
    subgraph clients [Clients]
        SDK[SDK / CLI]
        CI[CI Pipeline]
        Agent[AI Agent]
        UI[Dashboard]
    end

    subgraph dcp_control [DCP Control Plane]
        API[Intent & Transaction API]
        Auth[Domain OAuth / Capabilities]
        Compiler[Intent Compiler]
        Policy[Policy Engine]
        Kernel[Transactional Domain Kernel]
        Prov[Provenance Graph]
        Obs[Observability & Probes]
        AI[AI Planner - propose only]
        Sim[Simulator / Replay]
    end

    subgraph dcp_data [DCP Data Plane]
        Runtime[Route Runtime hosted/self]
        CertFW[Certificate Firewall]
        Immune[Takeover Immune System]
    end

    subgraph external [External Substrate]
        Reg[Registrars]
        DNS[Auth DNS Providers]
        CA[Certificate Authorities]
        Origins[Customer Origins]
    end

    clients --> API
    API --> Auth
    API --> Compiler
    Compiler --> Policy
    Policy --> Kernel
    Kernel --> Prov
    AI -.->|PlanProposal| Compiler
    Sim -.-> Compiler
    Kernel --> Obs

    Kernel --> RecipeRT[Recipe Runtime]
    RecipeRT --> Reg
    RecipeRT --> DNS

    Kernel --> Runtime
    Kernel --> CertFW
    CertFW --> CA
    Immune --> DNS
    Immune --> Runtime

    Runtime --> Origins
    DNS --> Runtime
```

---

## Component Responsibilities

| Component | Responsibility |
|-----------|----------------|
| Intent & Transaction API | CRUD intent, submit transactions, query status |
| Domain OAuth | Issue scoped capability tokens |
| Intent Compiler | Intent → IR → operation plan |
| Policy Engine | Deny/allow/audit rules on plans |
| Transactional Kernel | Lease, apply, verify, commit, rollback |
| Recipe Runtime | Execute signed provider adapters |
| Provenance Graph | Ownership, lineage, fences |
| Route Runtime | Instant HTTP/TLS routing |
| Certificate Firewall | Gate TLS issuance |
| Takeover Immune System | Detect dangling/reclaimable records |
| Observability | Probes, events, metrics, audit export |
| Simulator / Replay | Dry-run and historical re-execution |
| AI Planner | Natural language → PlanProposal (non-authoritative) |

---

## Request Lifecycle

```mermaid
sequenceDiagram
    participant C as Client
    participant API as DCP API
    participant Comp as Compiler
    participant Pol as Policy
    participant K as Kernel
    participant R as Recipe Runtime
    participant RT as Route Runtime

    C->>API: POST /transactions {intent_delta, idempotency_key}
    API->>Comp: compile(intent_v_next)
    Comp->>Pol: evaluate(plan)
    Pol-->>K: approved_plan
    K->>K: plan phase (no side effects)
    K->>K: acquire leases
    K->>R: apply operations
    R-->>K: attestations
    K->>K: verify (probes)
    K->>RT: push route config
    RT-->>K: routing_active
    K->>K: commit + provenance
    K-->>API: transaction committed
    API-->>C: {routing: active, dns: propagating, eta}
```

---

## State Stores

| Store | Contents | Consistency |
|-------|----------|-------------|
| Intent Store | Versioned intent documents | Strong (per domain) |
| Transaction Log | Append-only event sourcing | Strong |
| Provenance Graph | Nodes, edges, fences | Strong |
| Lease Table | Active resource locks | Strong, TTL |
| Route Config Store | Runtime routing versions | Strong + edge sync |
| Probe Results | Time-series propagation data | Eventual |
| Recipe Registry | Signed recipe bundles | Strong + CDN cache |

---

## Deployment Surfaces

See [dcp-04-deployment-modes.md](./dcp-04-deployment-modes.md).

| Mode | Control Plane | Route Runtime |
|------|---------------|---------------|
| Hosted SaaS | DCP cloud | DCP anycast edge |
| Hybrid | DCP cloud | Customer self-hosted runtime |
| Self-hosted | Customer VPC | Customer edge |
| Air-gapped | On-prem appliance | On-prem runtime |

---

## Failure Domains

| Domain | Blast radius | Mitigation |
|--------|--------------|------------|
| Single transaction | One domain resource | Rollback, lease expiry |
| Provider API outage | Affected operations paused | Retry, alternate recipe, degrade routing only |
| Control plane region loss | API unavailable | Multi-region failover; runtime serves last config |
| Runtime edge loss | Regional routing | Anycast failover, health checks |
| Compiler bug | Incorrect plans | Policy sandbox, simulation gate, version pinning |