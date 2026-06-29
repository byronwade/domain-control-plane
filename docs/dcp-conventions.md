# DCP Documentation Conventions

Standards for all Domain Control Plane (DCP) documentation in this repository.

---

## File Naming

```
docs/
  {category-folder}/
    dcp-{NN}-{kebab-case-slug}.md
```

| Rule | Example |
|------|---------|
| Prefix | Always `dcp-` |
| Number | Two-digit sequence within category (`01`, `02`, …) |
| Slug | Lowercase kebab-case, noun phrase |
| Extension | `.md` only |

**Exceptions:** `README.md`, `dcp-conventions.md`, `glossary/dcp-glossary.md`

### Category Folders

| Folder | Doc ID prefix | Sequence |
|--------|---------------|----------|
| `01-vision/` | `dcp-vision-` | Product & category |
| `02-architecture/` | `dcp-arch-` | System structure, speed, tech stack |
| `03-core-systems/` | `dcp-core-` | Engineering feats 01–10 |
| `04-apis/` | `dcp-api-` | HTTP/gRPC surface |
| `05-schemas/` | `dcp-schema-` | JSON Schema specs |
| `06-security/` | `dcp-sec-` | Threat model & controls |
| `07-roadmap/` | `dcp-roadmap-` | MVP & research |
| `08-examples/` | `dcp-example-` | Worked examples |
| `09-integrations/` | `dcp-integration-` | Third-party adapters |
| `10-reference/` | `dcp-ref-` | Catalogs & decision log |

---

## Document Header Template

Every `dcp-*.md` file begins with:

```markdown
# {Title}

| Field | Value |
|-------|-------|
| Doc ID | `dcp-{category}-{NN}` |
| Category | {Category Name} |
| Status | draft \| review \| stable |
| Version | 0.1.0-draft |
| Depends on | {comma-separated doc IDs or `none`} |
```

---

## Terminology

- Use terms from [glossary/dcp-glossary.md](./glossary/dcp-glossary.md) exactly.
- **DCP** = Domain Control Plane (the platform)
- **Intent** = declarative desired state (not raw DNS records)
- **Kernel** = transactional state machine that applies changes
- **Compiler** = intent → operation plan lowering pass
- **Recipe** = signed provider adapter script
- **Lease** = time-bounded exclusive mutation right on a resource
- **Provenance** = cryptographically linked change history

Do **not** use: "DNS replacement", "instant global DNS", "magic DNS".

---

## Section Order (Technical Docs)

1. Summary
2. Problem Statement
3. Design Goals & Non-Goals
4. Architecture / Data Model
5. Interfaces (API, events, schemas)
6. Algorithms & Invariants
7. Failure Modes & Recovery
8. Security Considerations
9. Observability
10. Open Questions (link to roadmap doc when resolved)

---

## Diagrams

- Prefer Mermaid for flow and sequence diagrams.
- Label trust boundaries explicitly.
- Distinguish **control plane** (DCP APIs) from **data plane** (DNS, HTTP routing, TLS termination).

---

## Cross-References

Link using relative paths:

```markdown
See [Transactional Domain Kernel](./03-core-systems/dcp-01-transactional-domain-kernel.md).
```

Reference doc IDs in prose: `dcp-core-01`.

---

## Honesty Principle

Every document that touches propagation or timing must state:

> DNS changes are eventually consistent globally. DCP optimizes for **instant routing for new requests** behind stable or proxied DNS, not instantaneous worldwide record visibility.