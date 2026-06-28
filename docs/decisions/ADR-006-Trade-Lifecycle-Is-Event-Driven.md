# ADR-006 — Trade Lifecycle Is Event-Driven

| Metadata | Value |
|----------|-------|
| **Status** | Accepted |
| **Date** | 2026-06-28 |
| **Decision Makers** | EdgeLab Project |
| **Category** | Architecture |

---

# Context

A trading position evolves through a sequence of observable events during its lifetime.

Examples include:

- Position creation
- Stop Loss modification
- Take Profit modification
- Partial close
- Feature recording
- Position closure

These events occur at discrete points in time and collectively describe the complete lifecycle of a trade.

The framework requires an interaction model capable of preserving this evolution while remaining independent of any specific trading platform.

---

# Decision

The Data Acquisition Framework adopts an event-driven interaction model.

The framework reacts to domain events describing changes in the lifecycle of a trade.

Typical interaction events include:

- Trade Opened
- Position Updated
- Feature Added
- Trade Closed

Platform Adapters are responsible for translating platform-specific events into this canonical event model.

---

# Rationale

An event-driven model naturally represents how trading activity evolves.

Each event captures a factual change that occurred during the lifecycle of a trade.

This approach aligns with the principle:

> Store Facts. Derive Everything Else.

Instead of periodically reconstructing trade state, the framework progressively builds the `TradeRecord` as new facts become available.

---

# Consequences

## Positive

- Clear separation between event detection and data acquisition.
- Chronological reconstruction of the trade lifecycle.
- Natural support for future timeline-based features.
- Platform-independent interaction model.

## Negative

- Platform Adapters must correctly identify and translate native platform events.
- Missing events may result in incomplete trade records.

---

# Alternatives Considered

## Polling-Based State Synchronization

Continuously reading the current state of trading positions was considered.

This option was rejected because it increases complexity, introduces unnecessary processing and may lose the temporal sequence of significant events.

The framework is designed to react to events rather than repeatedly inspect state.

---

# Related Documents

- Architecture Specification
- Integration Guide
- MQL5 Adapter Design
- ADR-002 — TradeRecord Represents Position Lifecycle
- ADR-005 — Store Facts, Derive Everything Else