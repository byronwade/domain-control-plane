# Revolutionary Experiences

| Field | Value |
|-------|-------|
| Doc ID | `dcp-vision-exp` |
| Category | Vision |
| Status | draft |
| Version | 0.1.0-draft |
| Created | 2026-06-28 |

---

## Goal

Define the specific experiences that should feel *revolutionary* to developers using DCP. These are the moments that should make DCP feel like a new category rather than an incremental improvement.

---

## Core Revolutionary Experiences

### 1. Instant Domain Changes
**Current pain**: Changing routing, compute, or configuration usually involves DNS propagation, waiting, or risk.

**Revolutionary version**: Most domain-level changes feel **instant** (target < 200ms p99). Developers change intent and see it live almost immediately, without thinking about DNS.

**Why it matters**: Removes one of the most annoying and unpredictable parts of infrastructure work.

### 2. True Instant Rollbacks
**Current pain**: Rollbacks are often slow, risky, or require manual intervention.

**Revolutionary version**: Rollback is **atomic and instant**. Pointing a domain back to a previous version takes effect immediately for new requests.

**Why it matters**: Developers can ship with much higher confidence and velocity.

### 3. Domain as a Programmable Unit
**Current pain**: Domains are usually just routing targets. Compute and state are managed separately.

**Revolutionary version**: A domain is a **programmable unit** where routing, compute (Programmable Edge), and state (Domain State Primitives) are versioned and managed together.

**Why it matters**: This feels like a new primitive — similar to how serverless changed how we think about compute.

### 4. Safe High-Velocity Iteration
**Current pain**: Fast changes at the edge often come with risk or weak safety.

**Revolutionary version**: Developers can iterate quickly with strong built-in safety (policy engine, provenance, versioning) and instant rollback if something goes wrong.

**Why it matters**: Combines speed with safety in a way that feels liberating.

### 5. Predictable Global Performance
**Current pain**: Tail latency can be unpredictable, especially globally.

**Revolutionary version**: Performance is highly consistent with excellent tail latency, supported by predictive pre-warming and smart routing.

**Why it matters**: The system feels reliable and responsive no matter where users are.

### 6. Intent-to-Execution with Safety
**Current pain**: Translating intent into working edge configuration is manual and error-prone.

**Revolutionary version**: Developers express high-level intent. DCP compiles it safely, with AI assistance that respects strong boundaries.

**Why it matters**: Makes complex domain operations feel simple and trustworthy.

---

## How These Experiences Connect

These experiences are not isolated. They reinforce each other:
- Instant changes + instant rollbacks = high velocity with safety.
- Domain as programmable unit + state primitives = powerful new patterns.
- Predictable performance + intent compilation = reliable global systems that are still easy to manage.

---

*This document defines the emotional and practical "wow" moments DCP should deliver.*