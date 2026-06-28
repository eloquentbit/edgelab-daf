# EdgeLab Data Acquisition Framework (DAF)

> **Transform trading data into knowledge.**

_A platform-independent framework for preserving trading facts as canonical datasets._

| Metadata              | Value            |
| --------------------- | ---------------- |
| **Project Status**    | Foundation Phase |
| **Architecture**      | Approved (v1.0)  |
| **Project Manifesto** | Approved (v1.0)  |
| **Current Version**   | 0.1.0            |

---

## The Problem

Every trading strategy produces trades.

Very few produce datasets suitable for quantitative research.

Most Expert Advisors eventually implement their own export logic, resulting in duplicated code, inconsistent formats and strategy-specific datasets that are difficult to analyze, compare and maintain.

Building a trading strategy should not require building a data engineering pipeline.

---

## The Solution

The EdgeLab Data Acquisition Framework (DAF) provides a canonical way to capture trade execution data and transform it into reliable datasets.

Instead of implementing custom export logic inside every Expert Advisor, trading strategies simply notify the framework about trade lifecycle events.

The framework progressively builds a complete `TradeRecord`, enriches it with additional features and exports the final dataset through interchangeable exporters.

The result is a clean separation between trading logic and data acquisition.

---

## Our Philosophy

We believe that better trading decisions start with better understanding.

Reliable knowledge starts with reliable facts.

Everything in the EdgeLab ecosystem exists to transform trading data into meaningful knowledge through engineering, reproducibility and scientific thinking.

---

## Why DAF?

The Data Acquisition Framework solves a problem that every quantitative developer eventually encounters.

Without a common acquisition layer, every strategy ends up reinventing the same infrastructure:

- trade recording
- CSV generation
- trade enrichment
- feature extraction
- dataset maintenance

DAF standardizes this process.

### Benefits

- **Write acquisition logic once** and reuse it across every strategy.
- **Produce canonical datasets** ready for quantitative analysis.
- **Keep trading strategies focused on trading**, not on data export.
- **Extend datasets through Features** without modifying the framework.
- **Export canonical datasets** through interchangeable exporters.
- **Integrate seamlessly with EdgeLab** while remaining completely independent from it.

---

> **Capture once. Analyze forever.**

---

## How It Works

The Data Acquisition Framework follows an event-driven architecture.

Trading strategies do not build datasets directly.

Instead, they notify the framework about trade lifecycle events.

The framework progressively builds a canonical `TradeRecord`, enriches it with strategy-specific features and finally exports it using one of the available exporters.

```text
Trading Strategy
        │
        ▼
 Platform Adapter
        │
        ▼
 DAF Core
        │
        ▼
    TradeRecord
        │
        ▼
     Exporter
        │
        ▼
 Canonical Dataset
```

This architecture keeps trading logic, data acquisition and persistence clearly separated while preserving a canonical representation of every trade.

---

## Vendor Neutral

The Data Acquisition Framework is completely independent from EdgeLab.

It produces canonical datasets that can be consumed by any quantitative analysis tool capable of consuming the exported datasets.

EdgeLab is the primary analysis engine built around DAF, but it is not a requirement.

---

## Why Open Source?

We believe that standardized trade datasets should not belong to a single platform.

By developing the Data Acquisition Framework as an open project, we hope to establish a common foundation for quantitative trading research that anyone can build upon.

The goal is not to build every possible analysis tool.

The goal is to establish a reliable foundation upon which better tools can be created.

---

## Documentation

The documentation is organized around the philosophy of the project.

We recommend starting with the Project Manifesto to understand why the project exists, then continuing with the Architecture Specification to understand how it is designed.

| Document                                        | Purpose                                           |
| ----------------------------------------------- | ------------------------------------------------- |
| `docs/branding/ProjectManifesto.md`             | Mission, vision and philosophy of the project.    |
| `docs/architecture/DataAcquisitionFramework.md` | Official architecture specification.              |
| `docs/api/PublicAPI.md`                          | Canonical interaction model.                       |
| `docs/guides/IntegrationGuide.md`                | Practical integration scenarios.                   |
| `ROADMAP.md`                                    | Planned milestones and future direction.          |
| `CONTRIBUTING.md`                               | Development workflow and contribution guidelines. |
| `CHANGELOG.md`                                  | Project history and released versions.            |

---

## Current Status

The project is currently in the **Foundation Phase**.

Completed:

- ✅ Project Manifesto v1.0
- ✅ Architecture Specification v1.0
- ✅ Repository foundation

Current focus:

- 🚧 Core implementation
- 🚧 MQL5 Platform Adapter
- 🚧 Initial unit tests

Our priority is to build a robust architectural foundation before implementing production-ready components.

---

## Roadmap

The planned evolution of the project is described in `ROADMAP.md`.
  
---

## License

This project is released under the Apache License 2.0.

See the `LICENSE` file for details.

---

> **Transform trading data into knowledge.**
