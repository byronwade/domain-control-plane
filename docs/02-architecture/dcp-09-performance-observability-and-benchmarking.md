# Performance Observability & Benchmarking

| Field | Value |
|-------|-------|
| Doc ID | `dcp-arch-09` |
| Category | Architecture |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-arch-07, dcp-core-03 |
| Created | 2026-06-28 |

---

## Summary

You cannot make DCP the fastest domain system without world-class observability and continuous benchmarking. This document defines the signals, dashboards, probing strategy, and benchmarking approach needed to hit (and continuously validate) the extreme speed SLOs.

It expands on the observability and benchmarking sections introduced in the extreme speed optimizations doc.

---

## Goals

- Real-time visibility into every P0/P1 operation (time-to-routing-active, propagation lag, cache behavior).
- Early detection of regressions in the fast path vs full saga path.
- Data-driven decisions when choosing between Pingora vs Envoy, Temporal vs Restate, etc.
- Public or customer-visible "speed scorecard" for marketing/positioning.

---

## Required Observability Signals

### 1. Distributed Tracing (OpenTelemetry)

Every transaction should carry a trace that covers:

- Intent submission (CLI or API)
- Compiler classification (fast path vs full path)
- Plan cache hit/miss
- Kernel fast-path execution (Tokio)
- Edge delta push + apply time
- Quorum ACK formation
- Version pointer swap
- First successful request on new bundle (from probe mesh)

**Key spans**:
- `dcp.compile`
- `dcp.kernel.fast_path`
- `dcp.edge.delta_push`
- `dcp.edge.quorum_ack`
- `dcp.pointer.swap`
- `dcp.probe.first_success`

### 2. Metrics (Prometheus / VictoriaMetrics)

High-value metrics (exemplars where cardinality is high):

| Metric | Labels | Purpose |
|--------|--------|---------|
| `dcp_route_accepted_seconds` | `tenant`, `fast_path` | Time from intent submit to accepted |
| `dcp_routing_active_seconds` | `tenant`, `bundle_version` | Time from commit to first edge traffic served |
| `dcp_edge_quorum_ack_seconds` | `pop_region` | How long to form quorum |
| `dcp_delta_apply_ms` | `pop`, `bundle_version` | Time to apply delta at edge |
| `dcp_plan_cache_hit_ratio` | `compiler_version` | Effectiveness of content-addressed cache |
| `dcp_propagation_lag_seconds` | `region`, `record_type` | DNS propagation from commit to global correctness |
| `dcp_tls_handshake_ms` | `cert_type` (wildcard vs per-host) | TLS performance |

### 3. Probe Mesh (ClickHouse time-series)

Global synthetic probes that query domains shortly after a commit and record:

- First successful response timestamp
- HTTP status + latency
- Which POP / resolver answered
- Bundle version seen in response header (`X-DCP-Bundle-Version`)

This gives accurate real-world propagation % and tail latency — far better than theoretical DNS TTL math.

**Implementation**: Lightweight probe agents (Rust or Go) deployed in major cloud regions + customer POPs. Results streamed to ClickHouse.

### 4. Audit & Provenance Events (NATS JetStream)

Every bundle version change, pointer swap, and rollback is emitted as an immutable event. These feed both security auditing and performance analysis ("how often do we roll back?").

---

## Dashboards & Alerts

### Primary Speed Dashboard

- Histogram of `time-to-routing-active` (target < 400ms p99)
- Heatmap of propagation lag by region
- Cache hit rate + reasons for misses
- Edge POP health + quorum formation success rate
- Fast-path vs full-path ratio (over time)

### Alerts (examples)

- p99 `routing_active_seconds` > 800ms for 5 minutes
- Propagation lag p95 > 90s for any region
- Plan cache hit rate drops below 60%
- Quorum formation fails > 1% of commits

---

## Benchmarking Strategy

### Philosophy
Benchmark against **realistic production-like conditions**, not just localhost. Include high route counts, mixed fast-path/full-path traffic, and global distribution.

### Tooling Recommendations

| Purpose | Recommended Tool | Notes |
|---------|------------------|-------|
| API / intent load | Custom Rust (tokio + Connect client) or k6 | Precise control over trace propagation |
| Sustained edge traffic | vegeta or custom pingora-based load gen | Measure p99 latency under load |
| Chaos / failure injection | Toxiproxy or custom gRPC proxy | Test quorum behavior and fast-path resilience |
| Profiling | `perf`, eBPF (Pixie or native), flamegraphs | Identify hot spots in Pingora / Hickory / compiler |
| DNS propagation | Custom probe mesh + public resolver queries | Real-world numbers vs lab |

### Phased Benchmark Plan

| Phase | Scope | Load Target | Key SLO Validation |
|-------|-------|-------------|--------------------|
| Prototype (weeks 1-6) | Local single POP, 5k–10k domains | Validate fast path < 400ms | Basic Tokio FSM + delta apply |
| MVP (weeks 7-16) | 3 POPs, 50k–100k domains, mixed traffic | End-to-end routing p99 < 1s | Dual-path kernel + Envoy Delta |
| Speed GA (weeks 17-28) | 8–12 global POPs, 500k–1M domains, high churn | Hit all P0/P1 SLOs at scale | Pingora + Hickory + plan cache |
| Vertical (weeks 29+) | 20+ POPs + self-hosted appliances | Sustained extreme load + chaos | Predictive pre-warming, NUMA, full observability |

All benchmark results should be stored in ClickHouse with git commit + config version tags for regression detection.

### Continuous Benchmarking in CI

- Nightly or on every `main` push: run lightweight probe suite against staging environment.
- On release candidates: full load + chaos suite.
- Automatic regression alerts in Slack / GitHub if any P0 SLO degrades > 10%.

---

## Implementation Priorities

1. **MVP**: Basic OpenTelemetry tracing + Prometheus metrics on the kernel and a simple edge mock.
2. **Speed phase**: Deploy global probe mesh + ClickHouse sink. Add exemplar sampling.
3. **GA**: Full dashboards, automated regression detection, public speed scorecard (optional).

---

## Related Documents

- [dcp-07-extreme-speed-optimizations.md](./dcp-07-extreme-speed-optimizations.md) — Source of the SLOs this document helps achieve
- [dcp-03-route-runtime.md](../03-core-systems/dcp-03-route-runtime.md) — Bundle version and delta mechanics that must be observable
- [dcp-08-technology-stack.md](./dcp-08-technology-stack.md) — Frameworks that expose good observability hooks

---

*This document turns performance from a goal into a measurable, continuously validated system property.*