# Public API
## Canonical Interaction Model

The Public API defines the canonical interaction model between a trading strategy and the Data Acquisition Framework (DAF).

Not every interaction represents a domain event.

Some interactions describe events that naturally occur during the lifecycle of a trading position, while others represent explicit requests from the trading strategy to enrich the resulting dataset.

For this reason, the Public API is divided into two conceptual categories: **Trade Lifecycle Events** and **Trade Enrichment Operations**.

### Trade Lifecycle Events

These events describe the beginning and the end of a trade.

| Event | Description |
|-------|-------------|
| **Trade Opened** | A new position has been successfully opened. |
| **Trade Closed** | The position has been completely closed and the trade lifecycle has ended. |

Lifecycle events are unique: a trade is opened once and closed once.

Together, these events define the canonical interaction model through which Platform Adapters communicate with the DAF Core.

### Trade Enrichment Operations

These operations allow a trading strategy to associate additional contextual information with the active `TradeRecord` without modifying its canonical lifecycle.

| Operation | Description |
|----------|-------------|
| **Feature Added** | Associates a custom feature with the active `TradeRecord`. |

Feature enrichment is intentionally separated from trade lifecycle events.

A feature does not modify the broker position.

Instead, it enriches the `TradeRecord` that the framework is progressively building.

This distinction preserves a clear separation of responsibilities:

- **Lifecycle events** describe what happened to the trading position.
- **Enrichment operations** describe additional contextual information recorded by the trading strategy.

This separation reinforces one of the fundamental design principles of the framework: trading strategies communicate facts, while the framework progressively builds a canonical TradeRecord that can later be transformed into knowledge by higher-level applications such as EdgeLab.

### Related Architecture Principles

The interaction model described in this document is derived from the architectural principles defined by the Architecture Specification and the Architecture Decision Records (ADR), particularly:

- ADR-002 — TradeRecord Represents Position Lifecycle
- ADR-004 — Platform Adapters Are External to the Core
- ADR-005 — Store Facts, Derive Everything Else
- ADR-006 — Trade Lifecycle Is Event-Driven

> **Note**
>
> At this stage, these are conceptual events rather than concrete method signatures.
> The actual API surface will be designed only after the event model has been fully validated.