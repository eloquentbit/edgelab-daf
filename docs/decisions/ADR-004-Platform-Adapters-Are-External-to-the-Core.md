# ADR-004 — Platform Adapters Are External to the Core

| Metadata | Value |
|----------|-------|
| **Status** | Accepted |
| **Date** | 2026-06-28 |
| **Decision Makers** | EdgeLab Project |
| **Category** | Architecture |

---

# Context

The Data Acquisition Framework is intended to support multiple trading platforms while maintaining a single canonical domain model.

Each trading platform exposes its own APIs, execution model and event lifecycle.

Examples include:

- MetaTrader 5
- MetaTrader 4
- cTrader
- TradeLocker
- Interactive Brokers
- Future supported platforms

Allowing platform-specific code to become part of the framework Core would tightly couple the architecture to a particular execution environment.

---

# Decision

Platform-specific integrations are implemented through dedicated Platform Adapters.

Platform Adapters are external to the DAF Core.

The Core defines the canonical interaction model and remains completely unaware of:

- platform APIs;
- platform callbacks;
- platform objects;
- platform-specific execution semantics.

The only responsibility of a Platform Adapter is to translate platform-specific events into the canonical interaction model understood by the Core.

---

# Rationale

Separating adapters from the Core establishes a clear architectural boundary.

The Core is responsible for acquiring, enriching and exporting trading knowledge.

Platform Adapters are responsible only for integration.

This separation allows both parts of the system to evolve independently while preserving a stable domain model.

---

# Consequences

## Positive

- The Core remains platform independent.
- Platform integrations evolve independently.
- New platforms can be supported without modifying the Core.
- Platform-specific complexity remains isolated.
- Testing the Core becomes significantly simpler.

## Negative

- Every supported platform requires its own adapter implementation.
- Adapter development requires platform-specific expertise.

---

# Alternatives Considered

## Platform Logic Inside the Core

Embedding platform-specific APIs directly inside the Core was considered.

This option was rejected because it would violate Platform Independence and significantly increase coupling.

---

## Platform-Specific Framework Versions

Maintaining separate framework implementations for different trading platforms was also considered.

This option was rejected because it would duplicate business logic and make long-term maintenance considerably more difficult.

The objective of the DAF is to maintain a single canonical Core shared by every supported platform.

---

# Related Documents

- Architecture Specification
- MQL5 Adapter Design
- Integration Guide
- ADR-001 — Platform Independence