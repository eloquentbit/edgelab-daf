# ADR-001 — Platform Independence

| Metadata | Value |
|----------|-------|
| **Status** | Accepted |
| **Date** | 2026-06-28 |
| **Decision Makers** | EdgeLab Project |
| **Category** | Architecture |

---

# Context

The Data Acquisition Framework is designed to become a platform-independent framework for acquiring structured trading data.

Trading platforms expose different execution models, APIs and event systems.

Coupling the framework to a specific platform would reduce portability, complicate maintenance and prevent future integrations.

The framework must therefore define its own canonical interaction model independently from any execution platform.

---

# Decision

The DAF Core shall remain completely independent from any trading platform.

Platform-specific concepts must never leak into the Core.

Every supported platform will integrate with the framework through a dedicated Platform Adapter responsible for translating native platform events into the canonical DAF interaction model.

The Core operates exclusively on this canonical model.

---

# Rationale

Separating platform integration from data acquisition provides several architectural benefits:

- Platform independence.
- Consistent `TradeRecord` generation.
- Reduced coupling.
- Easier testing.
- Independent evolution of adapters.
- Long-term maintainability.

---

# Consequences

## Positive

- New trading platforms can be supported without modifying the Core.
- The framework remains stable while adapters evolve independently.
- Testing becomes significantly easier because platform-specific APIs are isolated.

## Negative

- Every supported platform requires its own adapter implementation.
- Adapter development requires a deep understanding of the underlying trading platform.

---

# Alternatives Considered

## Platform-specific Core

Embedding platform-specific APIs directly inside the framework was considered.

This option was rejected because it would tightly couple the framework to a single execution environment and make future integrations considerably more difficult.

---

# Related Documents

- Architecture Specification
- MQL5 Adapter Design
- Integration Guide