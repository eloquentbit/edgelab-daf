# Integration Guide

| Metadata | Value |
|----------|-------|
| **Document Status** | Draft |
| **Target Audience** | Strategy Developers |
| **Document Version** | 0.1 |
| **Last Updated** | 2026-06-28 |
| **Author** | Luca Peduto |

---

# Purpose

This guide explains how trading strategies integrate with the EdgeLab Data Acquisition Framework (DAF).

Unlike the Architecture Specification, which describes the internal design of the framework, this document focuses on the developer experience.

Its purpose is to answer one simple question:

> **"How should my trading strategy interact with the DAF?"**

Rather than documenting implementation details, this guide presents practical integration scenarios that represent common trading workflows.

These scenarios serve as the primary validation tool for the framework's Public API.

If a scenario becomes difficult to implement or explain, the API should be reconsidered before implementation begins.

---

# Integration Philosophy

The Data Acquisition Framework follows a very simple philosophy:

> **Trading strategies generate trading events. The framework transforms those events into structured trading knowledge.**

A strategy should only communicate **facts**.

Examples include:

- A trade has been opened.
- A position has been updated.
- A trade has been closed.
- A new feature has been computed.

Everything else is handled internally by the framework.

The strategy is never responsible for:

- Building a `TradeRecord`
- Managing trade state
- Persisting datasets
- Exporting files
- Maintaining historical consistency

Those responsibilities belong exclusively to the DAF Core.

---

# Integration Scenarios

The following scenarios describe progressively more complex strategy integrations.

Every future evolution of the framework should preserve the simplicity of these workflows.

---

# Scenario 1 — Simple Trade

## Description

An Expert Advisor opens a market position.

The position remains unchanged until it is eventually closed.

## Trade Lifecycle

```text
Trade Opened
      │
      ▼
Trade Closed
```

## Strategy Enrichment

None.

## Expected Result

The framework creates a single `TradeRecord` containing the complete lifecycle of the position.

---

# Scenario 2 — Position Updates

## Description

An Expert Advisor opens a market position.

During its lifetime:

- Stop Loss is moved to Break Even.
- Trailing Stop updates the Stop Loss.
- The position is eventually closed.

## Trade Lifecycle

```text
Trade Opened
      │
      ▼
Position Updated
      │
      ▼
Position Updated
      │
      ▼
Trade Closed
```

## Strategy Enrichment

None.

## Expected Result

The active `TradeRecord` evolves during the lifetime of the position while preserving its identity.

---

# Scenario 3 — Strategy Enrichment

## Description

The strategy computes additional contextual information that should become part of the final dataset.

Examples include:

- ATR
- Opening Range Size
- Gap Percentage
- Session
- Market Regime
- Relative Volume

These values are attached to the active trade.

## Trade Lifecycle

```text
Trade Opened
      │
      ▼
Feature Added
      │
      ▼
Feature Added
      │
      ▼
Trade Closed
```

## Expected Result

The resulting `TradeRecord` contains:

- Standard Core Fields
- Strategy-specific Feature Set

without changing the canonical structure of the record.

---

# Scenario 4 — Partial Position Close *(Future)*

This scenario will document how the framework represents partial closes while maintaining a single canonical `TradeRecord`.

---

# Scenario 5 — Multiple Simultaneous Positions *(Future)*

This scenario will describe how multiple active trades are tracked independently.

---

# Scenario 6 — Pending Orders *(Future)*

This scenario will document how pending orders interact with the platform adapter and when they become actual trades.

---

# Scenario 7 — Multi-Symbol Strategies *(Future)*

This scenario will describe concurrent position management across multiple instruments.

---

# Scenario 8 — Cross-Platform Integration *(Future)*

This scenario will demonstrate that identical trading workflows can be implemented on different trading platforms through their respective Platform Adapters.

---

# Guiding Principles

Every new feature proposed for the Data Acquisition Framework should satisfy the following principles.

## Simplicity First

The integration workflow should remain straightforward and easy to understand.

---

## Facts Over Implementation

Trading strategies communicate facts.

The framework manages state.

---

## Platform Independence

Integration workflows should remain conceptually identical regardless of the execution platform.

---

## API Follows Workflows

The Public API should emerge naturally from these integration scenarios.

The scenarios define the desired developer experience.

The API is simply the mechanism used to achieve it.

---

# Conclusion

The Integration Guide represents the expected user experience of the Data Acquisition Framework.

It acts as a bridge between the Architecture Specification and the Public API, ensuring that the framework remains focused on simplicity, consistency and usability.

Every implementation decision should ultimately make these integration scenarios easier—not more complex.