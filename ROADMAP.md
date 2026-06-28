# Roadmap

The EdgeLab Data Acquisition Framework (DAF) is developed incrementally.

Each milestone represents a stable architectural evolution of the framework.

Rather than focusing on release dates, the roadmap is organized around capabilities that progressively build the foundation of the project.

---

# Current Status

**Current Version:** v0.1.0

Current focus:

- Architecture completed
- Documentation completed
- Core implementation about to begin

---

# v0.1.0 — Repository Foundation ✅

## Goals

Establish the architectural foundations of the project.

### Completed

- Project Manifesto
- Architecture Specification
- Public API
- Integration Guide
- MQL5 Adapter Design
- Architecture Decision Records (ADR)
- Open Source repository structure

Status: **Completed**

---

# v0.2.0 — Core Domain Model

## Goal

Implement the platform-independent core of the framework.

### Planned

- TradeRecord
- FeatureSet
- TradeRecorder
- Domain Events
- Initial Unit Tests

Outcome:

The framework can build and maintain canonical `TradeRecord` instances independently of any trading platform.

---

# v0.3.0 — Data Export

## Goal

Persist acquired trading knowledge.

### Planned

- CSV Exporter
- JSON Exporter
- Export configuration
- Dataset validation

Outcome:

TradeRecords can be exported into reusable datasets.

---

# v0.4.0 — MQL5 Platform Adapter

## Goal

Integrate the framework with MetaTrader 5.

### Planned

- Platform Adapter
- Trade lifecycle detection
- Position update handling
- Feature recording
- Example Expert Advisor

Outcome:

An MQL5 strategy can generate canonical TradeRecords through the DAF.

---

# v0.5.0 — Quality & Reliability

## Goal

Strengthen robustness before the first stable release.

### Planned

- Comprehensive Unit Tests
- Integration Tests
- Performance Validation
- Documentation Review
- Example Projects

Outcome:

The framework becomes production-ready for research purposes.

---

# v1.0.0 — First Stable Release

## Goal

Publish the first stable version of the Data Acquisition Framework.

### Planned

- Stable Public API
- Stable Core Architecture
- Stable MQL5 Adapter
- Complete Documentation
- Versioned Releases

Outcome:

A production-ready, platform-independent data acquisition framework for systematic trading research.

---

# Beyond v1.0

Future evolution may include:

- Additional Platform Adapters
  - MetaTrader 4
  - cTrader
  - TradeLocker
  - Interactive Brokers

- Additional Export Formats

- Plugin Ecosystem

- Community Contributions

- Integration with the broader EdgeLab ecosystem

---

# Guiding Principle

The roadmap prioritizes architectural stability over feature velocity.

Every new capability should reinforce the core philosophy of the project:

> **Store Facts. Derive Everything Else.**