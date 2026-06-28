# ADR-002 — TradeRecord Represents Position Lifecycle

| Metadata | Value |
|----------|-------|
| **Status** | Accepted |
| **Date** | 2026-06-28 |
| **Decision Makers** | EdgeLab Project |
| **Category** | Domain Model |

---

# Context

Different trading platforms expose different execution models.

For example, MetaTrader 5 distinguishes between:

- Orders
- Deals
- Positions

These concepts are specific to the platform and should not directly influence the internal domain model of the Data Acquisition Framework.

One of the primary objectives of the DAF is to provide a canonical and platform-independent representation of an executed trade.

To achieve this objective, the framework must determine which platform concept best represents the lifecycle that should be captured inside a `TradeRecord`.

---

# Decision

A `TradeRecord` represents the complete lifecycle of a trading **Position**.

It does **not** represent:

- an Order;
- a Deal;
- an individual platform transaction.

A `TradeRecord` is created when a position is opened, evolves while the position remains active and is finalized when the position is completely closed.

The `TradeRecord` therefore represents the canonical lifecycle of an executed trading position independently of the execution platform.

---

# Rationale

A Position naturally represents the business concept that traders usually consider to be a "trade".

Unlike Orders, Positions only exist after actual market execution.

Unlike Deals, a Position provides a continuous lifecycle that can evolve over time.

This allows the framework to capture events such as:

- Break Even activation;
- Stop Loss updates;
- Take Profit updates;
- Trailing Stop adjustments;
- Partial closes;
- Strategy-specific enrichment.

All these events belong to the same logical trade and therefore should contribute to a single canonical `TradeRecord`.

---

# Consequences

## Positive

- The framework exposes a platform-independent domain model.
- A single `TradeRecord` captures the complete evolution of a trade.
- Strategy features remain associated with one logical entity.
- Different trading platforms can generate identical `TradeRecord` structures through their respective Platform Adapters.

## Negative

- Platform Adapters must translate native execution models into the concept of a Position lifecycle.
- Some trading platforms may require additional logic to reconstruct the lifecycle of a Position.

---

# Alternatives Considered

## TradeRecord Represents an Order

This option was rejected because an Order represents an execution request rather than an executed trade.

Orders may never be executed, may expire or may be cancelled.

The DAF is interested in capturing executed trading activity.

---

## TradeRecord Represents a Deal

This option was rejected because multiple Deals may belong to the same logical trading Position.

Using Deals as the primary domain entity would fragment the representation of a single trade and make lifecycle analysis considerably more complex.

---

# Related Documents

- Architecture Specification
- MQL5 Adapter Design
- Integration Guide
- ADR-001 — Platform Independence