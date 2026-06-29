# DCP Glossary

| Field | Value |
|-------|-------|
| Doc ID | `dcp-glossary` |
| Category | Reference |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | none |

Canonical terminology for Domain Control Plane documentation.

---

## A

**Authoritative DNS** — The DNS zone where DCP (or a connected provider) holds write authority. Distinct from resolver caches and registrar glue.

**Apply Phase** — Kernel stage where compiled operations execute against providers under lease.

---

## B

**Blast Radius** — Maximum resources affected by a single transaction failure or policy violation.

---

## C

**Capability Token** — Scoped, short-lived credential authorizing specific domain mutations (e.g., `dns:write:api.example.com`).

**Certificate Firewall** — Policy engine that gates TLS issuance and renewal against provenance and takeover risk.

**Compile Phase** — Transformation of validated intent into an ordered, idempotent operation plan.

**Control Plane** — DCP APIs, kernel, compiler, policy, and observability. Does not terminate user traffic.

**CNAME Flattening** — Resolver/runtime behavior presenting apex records without exposing dangling CNAME chains.

---

## D

**Data Plane** — DNS responses, HTTP/TLS routing, and email delivery paths that serve end users.

**Domain Control Plane (DCP)** — Developer-native platform making domains programmable, permissioned, versioned, observable, and reversible.

**Domain Intent** — Declarative specification of desired domain behavior (routes, TLS, email auth, verifications) without raw record syntax.

**Domain OAuth** — OAuth 2.0 profile for delegating scoped domain capabilities to apps, CI, and agents.

**Domain Transaction** — Atomic unit of change with plan, apply, verify, and rollback semantics.

---

## E

**Effective Route** — Runtime routing decision for a request given current intent version and propagation state.

**Event Sourcing (Domain Log)** — Append-only log of intent versions, transactions, and provider effects.

---

## F

**Fence** — Provenance graph constraint preventing conflicting claims on the same FQDN.

---

## I

**Idempotency Key** — Client-supplied key ensuring duplicate transaction submissions collapse to one effect.

**Immune System** — Continuous monitor detecting dangling DNS, stale origins, and takeover conditions.

**Intent Compiler** — Deterministic lowering pipeline: intent → IR → operation plan → provider recipes.

**Intent IR** — Intermediate representation independent of any single DNS/hosting provider.

---

## K

**Kernel** — Transactional Domain Kernel; coordinates leases, phases, and rollback.

---

## L

**Lease** — Time-bounded lock on a domain resource during mutation; prevents split-brain writes.

**Lower** — Compiler action translating intent IR to provider-specific operations.

---

## O

**Operation Plan** — Ordered list of provider operations with dependencies, timeouts, and compensating actions.

**Ownership Graph** — Directed acyclic graph of who may assert control over zones, records, and routes.

---

## P

**Plan Phase** — Kernel stage producing operation plan without side effects (dry-run capable).

**Provenance** — Cryptographically linked metadata: who, what, when, why, and parent transaction.

**Provider Recipe** — Signed, sandboxed adapter describing how to mutate a specific provider (Cloudflare, Route53, Namecheap, etc.).

**Propagation Horizon** — Estimated time until DNS change reaches target percentile of resolvers globally.

---

## R

**Recipe Runtime** — Executes signed provider recipes with credential injection and output attestation.

**Replay** — Re-executing a historical transaction against a sandbox or shadow environment.

**Rollback** — Compensating transaction restoring prior intent version or safe degraded state.

**Route Runtime** — Hosted or self-hosted proxy/edge that applies routing rules instantly for new requests.

---

## S

**Simulation** — Offline evaluation of intent change against policy, compiler, and synthetic resolver models.

**Stable DNS** — Records pointing to DCP-controlled endpoints (proxy, anycast) that rarely change.

**Subdomain Takeover** — Attack where abandoned DNS records point to attacker-controlled resources.

---

## T

**Takeover Immune System** — Automated detection and remediation of dangling and reclaimable records.

**Transaction Phases** — `plan → lease → apply → verify → commit | rollback`.

---

## V

**Verify Phase** — Kernel stage confirming provider state, DNS propagation probes, TLS handshake, and route health.

**Version (Intent)** — Monotonic identifier for a domain's declared desired state.