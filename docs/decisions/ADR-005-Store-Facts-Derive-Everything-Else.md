# ADR-005 — Store Facts, Derive Everything Else

| Metadata | Value |
|----------|-------|
| **Status** | Accepted |
| **Date** | 2026-06-28 |
| **Decision Makers** | EdgeLab Project |
| **Category** | Data Model |

---

# Context

The primary objective of the Data Acquisition Framework is to produce datasets that remain useful over time.

Trading research continuously evolves.

New metrics, indicators and analytical techniques are regularly introduced.

If the framework stores only derived values, future analyses become limited by the assumptions made at the time of data acquisition.

The framework therefore requires a philosophy that maximizes the long-term value of collected trading data.

---

# Decision

The Data Acquisition Framework stores **facts**, not interpretations.

Every `TradeRecord` should contain the factual information describing what actually happened during the lifecycle of a trade.

Derived values should be computed later by analytics, research pipelines or higher-level applications such as EdgeLab.

Whenever possible:

- store observations;
- derive knowledge.

> *The DAF preserves information. EdgeLab transforms information into knowledge.*

---

# Rationale

Facts are stable.

Interpretations evolve.

For example:

- Entry Price is a fact.
- Exit Price is a fact.
- Stop Loss modification is a fact.
- ATR at entry is a fact.
- Session is a fact.

Conversely:

- Profit Factor
- Expectancy
- MAE
- MFE
- R Multiple
- Strategy Rating
- Trade Classification

are all interpretations that can be computed from the recorded facts.

By preserving factual information, the framework remains useful even when analytical methodologies evolve.

---

# Consequences

## Positive

- Historical datasets never become obsolete because of changing analytical methods.
- New metrics can be computed without reacquiring trading data.
- Multiple research methodologies can coexist using the same dataset.
- The framework remains focused on data acquisition rather than analytics.

## Negative

- Some derived metrics require additional processing after data acquisition.
- Exported datasets may contain more raw information than immediately required.

---

# Alternatives Considered

## Store Derived Metrics

Recording calculated metrics directly inside the canonical TradeRecord was considered.

This option was rejected because derived values depend on the analytical methodology available at a particular moment in time.

Embedding those calculations into the canonical data model would reduce flexibility and make historical datasets harder to reuse.

---

## Compute Everything During Acquisition

Performing analytical calculations while trades are being recorded was also considered.

This option was rejected because it mixes two distinct responsibilities:

- acquiring trading data;
- analyzing trading data.

The Data Acquisition Framework is responsible only for the first.

---

# Examples

The following examples illustrate the distinction.

| Store | Derive Later |
|--------|--------------|
| Entry Time | Holding Time |
| Exit Time | Trade Duration |
| Entry Price | R Multiple |
| Exit Price | Risk/Reward Ratio |
| Position Updates | MAE / MFE |
| ATR at Entry | ATR-based Statistics |
| Gap Percentage | Gap Analysis |
| Market Session | Session Performance |
| Commission | Net Performance |
| Swap | Carry Cost Analysis |

---

# Related Documents

- Architecture Specification
- Integration Guide
- ADR-002 — TradeRecord Represents Position Lifecycle
- ADR-003 — Core Fields and Feature Set