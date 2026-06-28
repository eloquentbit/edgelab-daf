# Source Code

This directory contains the implementation of the EdgeLab Data Acquisition Framework (DAF).

The framework is organized around a platform-independent Core and optional Platform Adapters that translate platform-specific concepts into the canonical interaction model understood by the DAF.

As the project evolves, this directory will contain the framework components responsible for:

- Building canonical `TradeRecord` instances.
- Recording trade lifecycle events.
- Managing strategy-specific feature enrichment.
- Exporting canonical trade datasets.

The implementation will evolve together with the architecture described in the project's documentation.