# EdgeLab Data Acquisition Framework (DAF)

> **Architecture Specification**

| Metadata                  | Value       |
| ------------------------- | ----------- |
| **Document Status**       | Approved    |
| **Implementation Status** | Planned     |
| **Document Version**      | 1.0         |
| **Last Updated**          | 2026-06-27  |
| **Author**                | Luca Peduto |

> This document defines the architecture of the EdgeLab Data Acquisition Framework (DAF).
>
> It serves as the authoritative reference for all implementation decisions related to the framework.

## Vision

The EdgeLab Data Acquisition Framework (DAF) is responsible for collecting accurate, complete and strategy-independent trading data from MQL5 Expert Advisors.

Its primary goal is to build a complete and accurate representation of every executed trade, preserving both its complete lifecycle and the contextual information required for quantitative analysis.

The generated dataset represents the only source of truth consumed by EdgeLab during quantitative analysis.

The framework must remain completely independent from any specific trading strategy.

## Goals

The framework has the following objectives:

- Collect only factual trading data.
- Produce a canonical trade dataset.
- Remain independent from any trading strategy.
- Support strategy-specific features.
- Separate data acquisition from data persistence.
- Be reusable across multiple Expert Advisors.
- Allow future export formats without changing trading code.

## Design Principles

The EdgeLab Data Acquisition Framework is designed around a small set of architectural principles that guide every implementation decision.

These principles ensure that the framework remains reusable, maintainable and independent from any specific trading strategy.

### 1. Strategy Agnostic

The framework must remain completely independent from any trading strategy.

It must not contain any knowledge about strategy-specific concepts such as Opening Range, ATR, RSI, EMA, Gap, Asian Session or any other trading logic.

Strategies provide information; the framework records it.

---

### 2. Store Facts, Derive Everything Else

The framework must record only factual information produced during trade execution.

Derived metrics and analytical calculations belong to EdgeLab and must never be computed inside the Data Acquisition Framework.

This guarantees that the dataset remains the single source of truth.

---

### 3. Separation of Responsibilities

The responsibilities of the system are clearly separated.

- Trading strategies make trading decisions.
- The Data Acquisition Framework captures trade events.
- Exporters persist collected data.
- EdgeLab performs quantitative analysis.

Each component must have one well-defined responsibility.

---

### 4. Platform Independence

The Data Acquisition Framework must remain completely independent from any specific trading platform.

Platform-specific integrations are implemented through dedicated Platform Adapters that translate native platform events into the framework interaction model.

As a consequence, the same canonical `TradeRecord` can be produced regardless of whether trades originate from MQL5, MQL4, cTrader, TradeLocker or any future supported platform.

This guarantees that trading knowledge remains portable even when execution platforms change.

---

### 5. Extensibility

New strategies must be able to enrich the trade dataset with additional features without requiring modifications to the framework.

The framework must provide a generic mechanism for collecting strategy-specific information.

---

### 6. Accuracy Before Convenience

Data accuracy always has higher priority than implementation simplicity.

Every recorded value must faithfully represent the actual execution performed by the trading platform.

No inferred or approximated value should replace an observable fact.

## High-Level Architecture

The Data Acquisition Framework acts as an intermediary layer between a trading strategy and the persistence layer.

Its responsibility is to transform platform-specific trading events into a standardized trade representation that can be consumed by EdgeLab.

The framework itself does not know anything about the trading strategy or the underlying trading platform.

```text
                    Trading Strategy
                           │
                           ▼
                   Platform Adapter
                           │
                           ▼
                        DAF Core
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                        ▼
          Core Fields                Features
              │                         │
              └────────────┬────────────┘
                           ▼
                      Trade Record
                           │
                           ▼
                        Exporter
                           │
                           ▼
                   Persistence Layer
```

This architecture clearly separates trading logic, data acquisition, and data persistence.

Each layer has a single responsibility and can evolve independently from the others.

## Components

The Data Acquisition Framework is composed of a small set of independent components.

Each component has a single responsibility and collaborates with the others to build a complete representation of an executed trade.

### Platform Adapter

The Platform Adapter is responsible for integrating a specific trading platform with the Data Acquisition Framework.

Its responsibility is to translate platform-specific events, objects and execution details into the framework interaction model.

The adapter acts as the only component that understands the underlying execution platform.

This architecture allows multiple independent adapters (e.g. MQL5, MQL4, cTrader, TradeLocker) to generate the same canonical `TradeRecord` without requiring changes to the framework core.

---

### Trade Recorder

The Trade Recorder manages the complete lifecycle of a trade.

It receives trade events, updates the current trade state, collects strategy features and produces the final `TradeRecord`.

It represents the central orchestration component of the DAF Core.

---

### Trade Record

The `TradeRecord` is the domain model representing a completed trade.

It contains all factual information collected during the trade lifecycle.

The `TradeRecord` contains no business logic and acts purely as a data model.

---

### Feature Collection

The Feature Collection is responsible for managing strategy-specific features associated with a trade.

The framework does not interpret feature values.

It simply stores them together with the standard trade information.

This allows every trading strategy to enrich the dataset without requiring changes to the framework itself.

---

### Exporter

The Exporter is responsible for persisting completed `TradeRecord` instances.

The Data Acquisition Framework does not depend on any specific persistence format.
The internal `TradeRecord` remains the canonical representation of a trade, while exporters are responsible for converting it into one or more persistence formats.

CSV represents the first supported exporter, while additional exporters (JSON, SQLite, Parquet, etc.) may be added in the future without affecting the framework architecture.

## Trade Lifecycle

The Data Acquisition Framework models the complete lifecycle of a trade.

A `TradeRecord` is progressively built as new information becomes available during trade execution.

The lifecycle is composed of the following stages:

```text
Trade Opened
      │
      ▼
Initialize Core Fields
      │
      ▼
Position Updated
      │
      ▼
Feature Added
      │
      ▼
Position Updated
      │
      ▼
Feature Added
      │
      ▼
Trade Closed
      │
      ▼
Finalize TradeRecord
      │
      ▼
Exporter
```

During its lifecycle, the `TradeRecord` remains under the exclusive responsibility of the Trade Recorder.

Once finalized, the record becomes immutable and is passed to the Exporter for persistence.

This guarantees a clear separation between trade acquisition and data persistence.

### Trade Evolution

During its lifetime, a trade may evolve multiple times before it is closed.

The framework distinguishes between two different kinds of information:

- **Broker events**, which describe factual changes to the underlying trading position.
- **Trade enrichment operations**, which associate additional strategy-specific knowledge with the current `TradeRecord`.

Both contribute to the progressive construction of the final `TradeRecord`, while preserving a strict separation between observable facts and contextual information.

## Trade Record

The `TradeRecord` represents the complete and immutable description of a single executed trade.

A `TradeRecord` represents the complete lifecycle of a trading Position independently of the underlying execution platform.

It is the central domain model of the Data Acquisition Framework and acts as the only unit exchanged between the acquisition layer and the persistence layer.

A `TradeRecord` is composed of two distinct parts:

- **Core Fields**
- **Feature Set**

```text
TradeRecord
│
├── Core Fields
│
├── Feature Set
│
└── Trade Timeline (Future)
```

The Trade Timeline is intentionally identified as a future evolution of the framework.

The first implementation focuses on building a complete canonical `TradeRecord`, while future versions may preserve the full sequence of trade lifecycle events and enrichment operations for richer quantitative analysis.

### Core Fields

Core Fields represent factual information that exists for every executed trade, regardless of the trading strategy.

Examples include:

- Strategy Name
- Strategy Version
- Symbol
- Position ID
- Open Time
- Close Time
- Entry Price
- Exit Price
- Stop Loss
- Take Profit
- Volume
- Profit
- Commission
- Swap

Core Fields are defined by the framework and remain stable across all trading strategies.

---

### Feature Set

The Feature Set contains strategy-specific information collected during the trade lifecycle.

Unlike Core Fields, Features are entirely defined by the trading strategy.

Examples include:

- Opening Range Size
- ATR
- RSI
- Gap Size
- EMA Distance
- Session Statistics

The framework stores Features without interpreting their meaning.

This allows every strategy to enrich the dataset while preserving the framework's strategy-independent design.

## Framework Interaction Model

The Data Acquisition Framework exposes a minimal interaction model rather than a large collection of public methods.

The interaction model is intentionally event-driven.

Trading strategies communicate observable facts while the framework manages the internal trade state.

Expert Advisors never manipulate the internal `TradeRecord` directly.

Instead, they communicate meaningful interactions that allow the framework to progressively build the canonical representation of a trade.

The interaction model is intentionally divided into two categories.

### Trade Lifecycle Events

These events originate from the trading platform and describe factual changes to the broker position.

```text
Trade Opened

↓

Position Updated

↓

Trade Closed
```

Lifecycle events describe observable facts and represent the evolution of the trading position.

### Trade Enrichment Operations

Trading strategies may optionally enrich the active `TradeRecord` with additional contextual information.

```text
Feature Added
```

Enrichment operations do not modify the broker position.

Instead, they extend the `TradeRecord` with strategy-specific knowledge that will become part of the final dataset.

This distinction preserves one of the fundamental architectural principles of the framework:

- Broker events describe facts.
- Enrichment operations describe knowledge.

The concrete Public API will be derived from this interaction model during implementation.

## Feature Model

Features represent strategy-specific information collected during the lifecycle of a trade.

Unlike Core Fields, which are defined by the framework, Features are entirely defined by the trading strategy.

Their purpose is to describe the context in which a trade was executed.

Examples of Features include:

- Opening Range Size
- ATR
- RSI
- Gap Size
- Market Regime
- Trading Session
- News Event
- Volatility Index

Features are intentionally strategy-independent.

The framework records Features without interpreting their meaning, validating their values or applying any business logic.

This design allows every trading strategy to enrich the dataset while preserving a stable and reusable framework architecture.

The Feature Model represents the primary extension mechanism of the Data Acquisition Framework.

## Future Extensions

The first version of the Data Acquisition Framework focuses on building a robust, strategy-independent trade acquisition layer.

The following extensions have already been identified as possible future evolutions of the framework:

- Multi-platform adapters.
- Additional exporters (JSON with trade timeline, SQLite, Parquet, REST API).
- Automatic Feature Providers.
- Feature validation and metadata.
- Strongly typed Features.
- Multi-position and portfolio recording.
- Real-time streaming exporters.
- Dataset versioning.

These features are intentionally excluded from the initial implementation.

The primary objective of the first release is to establish a simple, reliable and extensible architecture that can support future evolution without requiring fundamental redesign.

## Architectural Decisions

The following architectural decisions define the identity of the Data Acquisition Framework.

Each decision has been intentionally documented to preserve the rationale behind the architecture as the framework evolves.

| Decision | Reference |
|----------|-----------|
| Platform Independence | ADR-001 |
| TradeRecord Represents Position Lifecycle | ADR-002 |
| Core Fields and Feature Set | ADR-003 |
| Platform Adapters Are External to the Core | ADR-004 |
| Store Facts, Derive Everything Else | ADR-005 |
| Trade Lifecycle Is Event-Driven | ADR-006 |

The architectural rationale behind each decision is documented in the corresponding Architecture Decision Record (ADR).

Future architectural changes should preserve these principles or explicitly document the reasons for deviating from them.

## Conclusion

The Data Acquisition Framework establishes the architectural foundation of the EdgeLab ecosystem.

By separating platform integration from trade acquisition, the framework establishes a platform-independent standard for representing executed trades.

Execution platforms may evolve over time, but the resulting trading knowledge should remain portable, reusable and comparable across every supported environment.

Its primary objective is to provide a reliable, strategy-independent and extensible mechanism for producing canonical trade datasets that preserve trading facts independently of platform and strategy.

Future implementations should follow the principles described in this document while preserving the separation between data acquisition, persistence and quantitative analysis.
