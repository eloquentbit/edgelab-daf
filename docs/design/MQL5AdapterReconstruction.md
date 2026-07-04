# MQL5 Adapter Reconstruction

## 1. Purpose

This document defines the initial lifecycle reconstruction model for the EdgeLab DAF MQL5 Platform Adapter.

The reconstruction model describes how observable MetaTrader 5 execution facts are correlated and interpreted before they are translated into canonical EdgeLab DAF trade data.

The specification is derived from empirical platform observations documented in:

```text
research/mql5/TradeTransactionProbe.md
```

This document is intentionally limited to behavior that has been observed and supported by experimental evidence.

It does not attempt to provide a complete abstraction of every MetaTrader 5 trading operation.

---

## 2. Scope

The initial reconstruction model covers:

- executed Deals observed through `TRADE_TRANSACTION_DEAL_ADD`;
- lifecycle correlation using `DEAL_POSITION_ID`;
- entry execution accumulation;
- exit execution accumulation;
- partial exit reconstruction;
- lifecycle completion reconstruction;
- execution cost accumulation;
- optional enrichment from current Position state.

The initial model is based on observations from:

```text
ACCOUNT_MARGIN_MODE_RETAIL_HEDGING
```

The following behaviors remain outside the initial implementation scope:

- netting-account reconstruction;
- position reversals;
- `DEAL_ENTRY_INOUT`;
- `DEAL_ENTRY_OUT_BY`;
- close-by operations;
- pending-order lifecycle reconstruction;
- rejected requests;
- cancelled orders;
- partial order fills;
- multi-Deal order execution;
- Stop Out semantics.

Unsupported behavior must not be silently interpreted using unverified assumptions.

---

## 3. Design Principle

The MQL5 Platform Adapter reconstructs trade lifecycles from execution evidence.

The primary reconstruction flow is:

```text
MetaTrader 5 Deal
        |
        v
Extract execution facts
        |
        v
Identify lifecycle
        |
        v
Accumulate lifecycle evidence
        |
        v
Evaluate lifecycle state
        |
        v
Produce canonical trade data
```

The Adapter must not infer strategy intent from platform activity.

For example:

```text
Same-direction entry
```

must not automatically become:

```text
Scale In
```

The platform evidence and the strategy interpretation are separate concerns.

---

## 4. Primary Execution Evidence

The initial MQL5 Adapter treats:

```text
TRADE_TRANSACTION_DEAL_ADD
```

as the primary signal that a new execution fact is available.

When such a transaction is observed, the Adapter retrieves the associated Deal from MetaTrader 5 history.

The Deal provides the execution evidence used for lifecycle reconstruction.

Relevant Deal properties include:

```text
DEAL_TICKET
DEAL_ORDER
DEAL_POSITION_ID
DEAL_TIME
DEAL_TIME_MSC
DEAL_TYPE
DEAL_ENTRY
DEAL_REASON
DEAL_MAGIC
DEAL_SYMBOL
DEAL_VOLUME
DEAL_PRICE
DEAL_COMMISSION
DEAL_SWAP
DEAL_PROFIT
DEAL_COMMENT
```

Not every property is necessarily part of the final canonical trade model.

Platform-specific data may be used internally during reconstruction without being exposed through the public EdgeLab DAF API.

---

## 5. Lifecycle Correlation

### 5.1 Correlation Key

For the initial hedging-account implementation, the lifecycle correlation key is:

```text
DEAL_POSITION_ID
```

All supported execution Deals sharing the same `DEAL_POSITION_ID` are treated as evidence belonging to the same MQL5 Position lifecycle.

Conceptually:

```text
DEAL_POSITION_ID = 42
        |
        +-- Deal A
        |
        +-- Deal B
        |
        +-- Deal C
        |
        v
Reconstructed lifecycle 42
```

### 5.2 Why Symbol Is Insufficient

Multiple independent Positions may exist for the same symbol under the hedging accounting model.

Therefore:

```text
symbol
```

cannot identify a lifecycle.

### 5.3 Why Symbol and Magic Are Insufficient

Multiple independent Positions may also share:

```text
symbol
magic
```

Therefore:

```text
symbol + magic
```

must not be used as the lifecycle correlation key.

Symbol and magic number remain metadata and filtering attributes.

### 5.4 Correlation Scope

The initial correlation rule applies only to the empirically observed hedging-account behavior.

It must not be generalized to netting-account reconstruction without separate verification.

---

## 6. Execution Classification

The Adapter classifies supported Deals using `DEAL_ENTRY`.

### 6.1 Entry Execution

A Deal with:

```text
DEAL_ENTRY_IN
```

is classified as an entry execution.

The execution contributes:

```text
DEAL_VOLUME
```

to accumulated entry volume.

The Deal price contributes to entry-price reconstruction.

The Deal costs contribute to lifecycle cost accumulation.

### 6.2 Exit Execution

A Deal with:

```text
DEAL_ENTRY_OUT
```

is classified as an exit execution.

The execution contributes:

```text
DEAL_VOLUME
```

to accumulated exit volume.

The Deal price contributes to exit-price reconstruction.

The Deal profit and costs contribute to lifecycle financial reconstruction.

### 6.3 Unsupported Entry Types

The initial implementation does not interpret:

```text
DEAL_ENTRY_INOUT
DEAL_ENTRY_OUT_BY
```

These Deal entry types require separate empirical investigation.

The Adapter must detect unsupported entry types explicitly.

They must not be silently classified as ordinary entry or exit executions.

---

## 7. Lifecycle Accumulation

For each lifecycle correlation key, the Adapter maintains accumulated execution evidence.

Conceptually:

```text
Lifecycle
---------
position_id
entry_volume
exit_volume
entry_value
exit_value
commission
swap
realized_profit
first_entry_time
last_exit_time
```

This is an internal reconstruction model.

It is not the canonical `TradeRecord`.

The internal lifecycle state exists only to transform platform execution evidence into canonical trade data.

---

## 8. Entry Volume Reconstruction

For every supported Deal where:

```text
DEAL_ENTRY = DEAL_ENTRY_IN
```

the Adapter performs:

```text
entry_volume += DEAL_VOLUME
```

Example:

```text
Deal A
DEAL_ENTRY_IN
volume = 0.10

Deal B
DEAL_ENTRY_IN
volume = 0.05
```

Reconstructed entry volume:

```text
entry_volume = 0.15
```

This rule describes execution accumulation.

It does not imply that the strategy intended a scale-in operation.

---

## 9. Exit Volume Reconstruction

For every supported Deal where:

```text
DEAL_ENTRY = DEAL_ENTRY_OUT
```

the Adapter performs:

```text
exit_volume += DEAL_VOLUME
```

Example:

```text
Deal A
DEAL_ENTRY_OUT
volume = 0.05

Deal B
DEAL_ENTRY_OUT
volume = 0.05
```

Reconstructed exit volume:

```text
exit_volume = 0.10
```

An individual exit Deal must not independently be interpreted as lifecycle completion.

---

## 10. Lifecycle Exposure

The reconstructed lifecycle exposure is derived from execution volume:

```text
remaining_volume = entry_volume - exit_volume
```

The Adapter must account for platform volume precision when comparing reconstructed volumes.

Conceptually:

```text
entry_volume = 0.10
exit_volume  = 0.05

remaining_volume = 0.05
```

The lifecycle remains open.

If:

```text
entry_volume = 0.10
exit_volume  = 0.10

remaining_volume = 0.00
```

the lifecycle is complete.

Current Position availability may be used as supporting evidence but is not required for this calculation.

---

## 11. Lifecycle Completion

A supported lifecycle is considered complete when reconstructed execution evidence indicates that no executed exposure remains.

Conceptually:

```text
entry_volume > 0
```

and:

```text
remaining_volume == 0
```

within the applicable volume precision.

Therefore:

```text
DEAL_ENTRY_OUT
```

does not mean:

```text
Lifecycle Complete
```

Instead:

```text
DEAL_ENTRY_OUT
        |
        v
Accumulate exit volume
        |
        v
Recalculate remaining volume
        |
        v
remaining volume == 0 ?
        |
    +---+---+
    |       |
   NO      YES
    |       |
    v       v
 Open    Complete
```

Lifecycle completion is a reconstructed fact.

It is not derived from the existence of a single exit Deal.

---

## 12. Current Position State

Current MetaTrader 5 Position state is optional reconstruction evidence.

It may provide useful live information such as:

```text
POSITION_VOLUME
POSITION_PRICE_OPEN
POSITION_PRICE_CURRENT
POSITION_SL
POSITION_TP
POSITION_PROFIT
```

However, current Position state must not be required for historical lifecycle reconstruction.

A fully closed Position may no longer be selectable when its final Deal is processed.

Therefore, logic equivalent to:

```mql5
if(!PositionSelectByTicket(position_ticket))
   return;
```

must not prevent Deal-based lifecycle reconstruction.

Current Position state may enrich a lifecycle.

It must not define whether historical execution evidence exists.

---

## 13. Operational Position Mutations

Operational Position mutations are distinct from execution evidence.

The observed example is:

```text
TRADE_ACTION_SLTP
```

A Stop Loss or Take Profit modification may change:

```text
POSITION_SL
POSITION_TP
```

without creating a Deal.

Therefore, operational mutations must not modify:

```text
entry_volume
exit_volume
entry execution history
exit execution history
```

The initial reconstruction model does not require SL/TP mutation history to construct canonical trade execution facts.

Whether operational state history becomes a separate future DAF capability is outside the scope of this specification.

---

## 14. Price Reconstruction

Entry and exit prices must be reconstructed from executed Deal prices.

### 14.1 Entry Price

For one entry Deal:

```text
entry_price = DEAL_PRICE
```

For multiple entry Deals belonging to the same lifecycle, the reconstructed entry price is volume weighted.

Conceptually:

```text
entry_value += DEAL_PRICE * DEAL_VOLUME
```

Then:

```text
entry_price = entry_value / entry_volume
```

### 14.2 Exit Price

For one exit Deal:

```text
exit_price = DEAL_PRICE
```

For multiple exit Deals belonging to the same lifecycle, the reconstructed exit price is volume weighted.

Conceptually:

```text
exit_value += DEAL_PRICE * DEAL_VOLUME
```

Then:

```text
exit_price = exit_value / exit_volume
```

The Adapter must derive execution prices from Deals rather than from current Position prices.

---

## 15. Financial Accumulation

The Adapter accumulates financial execution facts from correlated Deals.

Conceptually:

```text
commission += DEAL_COMMISSION
swap       += DEAL_SWAP
profit     += DEAL_PROFIT
```

The initial reconstruction model preserves the platform-reported values.

The Adapter must not recalculate broker-reported realized profit from price movement when the corresponding Deal financial data is available.

Platform-reported execution facts are the primary evidence.

---

## 16. Time Reconstruction

The lifecycle entry time is derived from entry execution evidence.

Conceptually:

```text
entry_time = earliest supported DEAL_ENTRY_IN execution time
```

The lifecycle exit time is derived from exit execution evidence.

For a completed lifecycle:

```text
exit_time = latest supported DEAL_ENTRY_OUT execution time
```

Where available, millisecond precision should be preserved internally using:

```text
DEAL_TIME_MSC
```

The canonical representation may normalize timestamp precision according to the EdgeLab DAF public data contract.

---

## 17. Processing Independence from Callback Order

The reconstruction result must depend on Deal evidence, not on the order in which `OnTradeTransaction()` callbacks are received.

The following callback sequence was empirically observed:

```text
TRADE_TRANSACTION_DEAL_ADD
TRADE_TRANSACTION_ORDER_DELETE
TRADE_TRANSACTION_HISTORY_ADD
TRADE_TRANSACTION_REQUEST
```

The Adapter must not require:

```text
REQUEST
```

to be processed before:

```text
DEAL_ADD
```

Lifecycle reconstruction must remain valid when execution evidence is processed independently of request callback ordering.

---

## 18. Duplicate Deal Protection

A Deal represents immutable execution evidence.

The Adapter must not apply the same Deal more than once to a reconstructed lifecycle.

The initial implementation must therefore track processed Deal identifiers using:

```text
DEAL_TICKET
```

Conceptually:

```text
Deal received
     |
     v
Already processed?
     |
 +---+---+
 |       |
YES      NO
 |       |
 v       v
Ignore   Apply
```

Duplicate protection is required even if normal platform behavior usually delivers one `TRADE_TRANSACTION_DEAL_ADD` callback per added Deal.

Reconstruction correctness must not depend on accidental duplicate-free ingestion.

---

## 19. Initial Reconstruction Algorithm

The initial supported reconstruction flow is:

```text
OnTradeTransaction
        |
        v
Is transaction DEAL_ADD?
        |
    +---+---+
    |       |
   NO      YES
    |       |
    v       v
 Ignore   Load Deal
              |
              v
       Deal already processed?
              |
          +---+---+
          |       |
         YES      NO
          |       |
          v       v
        Ignore   Read DEAL_POSITION_ID
                      |
                      v
              Find or create lifecycle
                      |
                      v
                Read DEAL_ENTRY
                      |
              +-------+-------+
              |               |
        DEAL_ENTRY_IN   DEAL_ENTRY_OUT
              |               |
              v               v
       Accumulate entry   Accumulate exit
              |               |
              +-------+-------+
                      |
                      v
             Accumulate financial facts
                      |
                      v
             Mark Deal as processed
                      |
                      v
             Recalculate exposure
                      |
                      v
              Evaluate completion
```

Unsupported `DEAL_ENTRY` values must follow an explicit unsupported-data path.

They must not be coerced into the supported entry or exit branches.

---

## 20. Reconstruction Invariants

The initial MQL5 reconstruction implementation should preserve the following invariants.

### I1 — A Deal Is Applied at Most Once

The same `DEAL_TICKET` must never contribute execution volume or financial values more than once.

### I2 — Lifecycle Correlation Uses Position ID

Supported Deals are correlated using `DEAL_POSITION_ID`.

### I3 — Entry Volume Comes Only from Entry Executions

Only supported `DEAL_ENTRY_IN` Deals contribute to `entry_volume`.

### I4 — Exit Volume Comes Only from Exit Executions

Only supported `DEAL_ENTRY_OUT` Deals contribute to `exit_volume`.

### I5 — Operational Mutations Do Not Change Execution Volume

SL/TP modifications must not affect reconstructed entry or exit volume.

### I6 — Lifecycle Completion Is Reconstructed

A lifecycle is complete only when supported execution evidence indicates no remaining executed exposure.

### I7 — Current Position Availability Is Not Required

The absence of a selectable current Position must not invalidate historical Deal reconstruction.

### I8 — Strategy Intent Is Not Inferred

The Adapter must not infer concepts such as scale-in from same-direction execution activity alone.

---

## 21. Unsupported Evidence

When the Adapter encounters execution evidence outside the supported reconstruction model, it must not silently guess.

Examples include:

```text
DEAL_ENTRY_INOUT
DEAL_ENTRY_OUT_BY
```

The exact runtime behavior will be defined during implementation.

Possible handling strategies include:

```text
diagnostic reporting
explicit unsupported status
deferred reconstruction
```

Silent semantic coercion is not acceptable.

For example:

```text
DEAL_ENTRY_INOUT
```

must not automatically be treated as:

```text
DEAL_ENTRY_OUT
```

or:

```text
DEAL_ENTRY_IN
```

without verified platform-specific reconstruction rules.

---

## 22. Relationship to the Canonical Model

The reconstruction lifecycle is platform-specific internal state.

It exists between:

```text
MetaTrader 5
```

and:

```text
EdgeLab DAF canonical trade model
```

Conceptually:

```text
MQL5 Deal History
        |
        v
MQL5 Lifecycle Reconstruction
        |
        v
Canonical Trade Mapping
        |
        v
TradeRecord
```

The reconstruction model must not leak MQL5-specific concepts into the canonical public model unless they are explicitly preserved as platform metadata.

Examples of MQL5-specific reconstruction details include:

```text
DEAL_TICKET
DEAL_POSITION_ID
DEAL_ENTRY
```

These properties may be necessary to reconstruct the trade without becoming first-class canonical trade fields.

---

## 23. Design Boundaries

The MQL5 Platform Adapter is responsible for:

```text
observing MQL5 execution evidence
correlating supported Deals
reconstructing Position lifecycles
detecting lifecycle completion
mapping reconstructed facts to canonical data
```

The Adapter is not responsible for:

```text
inferring strategy intent
classifying scale-in decisions
evaluating trade quality
calculating strategy statistics
persisting research experiments
performing portfolio analysis
```

These responsibilities belong to other EdgeLab layers or downstream research tooling.

---

## 24. Conclusion

The initial MQL5 Adapter reconstruction model is Deal-centric.

Supported execution Deals are correlated using `DEAL_POSITION_ID` and accumulated into platform-specific lifecycle state.

Entry and exit volume, execution prices, financial facts, and lifecycle completion are reconstructed from immutable Deal evidence.

Current Position state is optional enrichment and is not required for historical reconstruction.

Operational Position mutations are separated from execution facts.

The Adapter does not infer strategy intent from platform activity.

This specification defines the minimum reconstruction behavior required before implementing the first MQL5 runtime components.

The next implementation step is to define the minimal internal MQL5 lifecycle data structure that can preserve these invariants without prematurely implementing the canonical `TradeRecord`.