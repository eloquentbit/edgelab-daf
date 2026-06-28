# Contributing

First of all, thank you for your interest in contributing to the EdgeLab Data Acquisition Framework (DAF).

Whether you are reporting a bug, suggesting a new idea or contributing code, your help is greatly appreciated.

The goal of this document is not only to explain **how** to contribute, but also **how architectural decisions are made** within the project.

---

# Before You Start

Before contributing, we strongly recommend reading the following documents:

1. README
2. Project Manifesto
3. Architecture Specification
4. Integration Guide
5. Architecture Decision Records (ADR)

These documents describe the philosophy, architecture and long-term vision of the project.

Understanding them before writing code will make contributions significantly more effective.

---

# Our Philosophy

The Data Acquisition Framework values:

- Simplicity over cleverness.
- Architectural consistency over implementation speed.
- Facts over interpretations.
- Platform independence.
- Long-term maintainability.

Every contribution should reinforce these principles.

---

# Development Workflow

When contributing new functionality, we encourage the following workflow:

```text
Discussion
    │
    ▼
Architecture (if necessary)
    │
    ▼
Documentation
    │
    ▼
Implementation
    │
    ▼
Testing
```

For significant architectural changes, implementation should never be the first step.

---

# Documentation First

Documentation is considered part of the implementation, not an afterthought.

When introducing significant functionality, contributors are encouraged to update the relevant documentation before—or together with—the implementation.

This includes, when appropriate:

- Architecture Specification
- Integration Guide
- Public API
- MQL5 Adapter Design
- Architecture Decision Records (ADR)

A feature is considered complete only when both the implementation and its documentation are consistent.

---

# Keep the Core Small

One of the primary design goals of the DAF is to keep the Core as small and stable as possible.

Before introducing a new class, ask yourself:

- Does this solve a real problem?
- Can the same result be achieved with a simpler design?
- Does this belong in the Core or inside a Platform Adapter?

Every abstraction has a maintenance cost.

New abstractions should exist only when they provide clear architectural value.

---

# Architecture Decisions

Major architectural changes should be discussed before implementation.

When appropriate, new Architecture Decision Records (ADR) should be proposed.

ADRs preserve the reasoning behind important decisions and help maintain long-term consistency across the project.

---

# Coding Principles

The project favors:

- Readable code.
- Small and focused components.
- Explicit naming.
- Clear responsibilities.
- Composition over unnecessary complexity.

Code should be easy to understand before it is optimized.

---

# Pull Requests

Good pull requests are:

- Small.
- Focused.
- Well documented.
- Easy to review.

Whenever possible, include:

- Motivation.
- Design considerations.
- Documentation updates.
- Tests.

---

# Reporting Issues

Bug reports should include:

- Platform
- Framework version
- Steps to reproduce
- Expected behavior
- Actual behavior

Feature requests should focus on the problem being solved rather than a specific implementation.

---

# Guiding Principle

The purpose of the Data Acquisition Framework is not to collect as many features as possible.

Its purpose is to provide a simple, robust and platform-independent foundation for acquiring structured trading knowledge.

Whenever a design decision is uncertain, prefer the solution that preserves simplicity.

> **Store Facts. Derive Everything Else.**

Thank you for helping build EdgeLab DAF.