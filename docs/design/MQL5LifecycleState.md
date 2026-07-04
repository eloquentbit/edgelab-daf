

# MQL5 Lifecycle State

## 1. Purpose

This document defines the minimal internal lifecycle state used by the EdgeLab DAF MQL5 Platform Adapter during Deal-based reconstruction.

The lifecycle state accumulates platform execution facts belonging to one supported MQL5 Position lifecycle.

It is an internal Adapter concept.

It is not:

- a MetaTrader 5 Position snapshot;
- a canonical `TradeRecord`;
- a strategy-level trade model;
- a persistence model.

The lifecycle state exists only to preserve the minimum evidence required to reconstruct a supported lifecycle deterministically.

---

## 2. Design Context

The reconstruction model is defined in:

```text
 docs/design/MQL5AdapterReconstruction.md
```

The model is based on empirical observations documented in:

```text
research/mql5/TradeTransactionProbe.md
```

The reconstruction flow is:

```text
MQL5 Deal evidence
        |
        v
Mql5LifecycleState
        |
        v
Reconstructed lifecycle facts
        |
        v
Canonical trade mapping
```

`Mql5LifecycleState` represents the accumulated state of one lifecycle correlation key.

For the initial hedging-account implementation, that correlation key is:

```text
DEAL_POSITION_ID
```

---

## 3. Responsibility

`Mql5LifecycleState` is responsible for preserving accumulated lifecycle facts derived from supported execution Deals.

Its responsibilities are limited to:

- identifying the correlated MQL5 Position lifecycle;
- preserving stable lifecycle metadata;
- accumulating entry execution volume and value;
- accumulating exit execution volume and value;
- accumulating platform-reported financial facts;
- preserving lifecycle execution boundaries in time;
- exposing deterministic derived lifecycle values.

The state object must remain independent from callback ordering and current Position availability.

---

## 4. Stored State

The minimal stored state is:

```text
position_id
symbol
magic
direction
entry_volume
exit_volume
entry_value
exit_value
commission
swap
realized_profit
first_entry_time_msc
last_exit_time_msc
```

Each field preserves accumulated or stable lifecycle evidence.

### 4.1 Position ID

```text
position_id
```

stores the MQL5 lifecycle correlation identifier derived from:

```text
DEAL_POSITION_ID
```

The value identifies the lifecycle represented by the state object.

It is immutable after state construction.

### 4.2 Symbol

```text
symbol
```

stores the symbol associated with the lifecycle execution evidence.

Symbol is lifecycle metadata.

It is not used as the lifecycle correlation key.

The value is established from supported entry execution evidence and must remain consistent for the lifecycle.

### 4.3 Magic Number

```text
magic
```

stores the MQL5 magic number associated with the lifecycle execution evidence.

Magic number is lifecycle metadata.

It is not used as the lifecycle correlation key.

The value is established from supported entry execution evidence and must remain consistent for the lifecycle.

### 4.4 Direction

```text
direction
```

stores the reconstructed lifecycle direction.

Direction is derived from supported entry execution evidence.

The internal direction model is:

```text
UNKNOWN
LONG
SHORT
```

Conceptually:

```text
DEAL_ENTRY_IN + DEAL_TYPE_BUY
        -> LONG

DEAL_ENTRY_IN + DEAL_TYPE_SELL
        -> SHORT
```

Exit Deal direction must not redefine lifecycle direction.

For example, a SELL Deal with:

```text
DEAL_ENTRY_OUT
```

may represent an exit from a LONG lifecycle.

### 4.5 Entry Volume

```text
entry_volume
```

stores accumulated executed entry volume.

For every supported entry Deal:

```text
entry_volume += DEAL_VOLUME
```

### 4.6 Exit Volume

```text
exit_volume
```

stores accumulated executed exit volume.

For every supported exit Deal:

```text
exit_volume += DEAL_VOLUME
```

### 4.7 Entry Value

```text
entry_value
```

stores accumulated volume-weighted entry execution value.

For every supported entry Deal:

```text
entry_value += DEAL_PRICE * DEAL_VOLUME
```

The value exists to derive the volume-weighted entry price.

### 4.8 Exit Value

```text
exit_value
```

stores accumulated volume-weighted exit execution value.

For every supported exit Deal:

```text
exit_value += DEAL_PRICE * DEAL_VOLUME
```

The value exists to derive the volume-weighted exit price.

### 4.9 Commission

```text
commission
```

stores accumulated platform-reported commission.

For every supported Deal:

```text
commission += DEAL_COMMISSION
```

### 4.10 Swap

```text
swap
```

stores accumulated platform-reported swap.

For every supported Deal:

```text
swap += DEAL_SWAP
```

### 4.11 Realized Profit

```text
realized_profit
```

stores accumulated platform-reported Deal profit.

For every supported Deal:

```text
realized_profit += DEAL_PROFIT
```

The lifecycle state preserves the platform-reported value.

It does not independently recalculate realized profit from price movement.

### 4.12 First Entry Time

```text
first_entry_time_msc
```

stores the earliest supported entry execution time in milliseconds.

The value is derived from:

```text
DEAL_TIME_MSC
```

for supported `DEAL_ENTRY_IN` Deals.

### 4.13 Last Exit Time

```text
last_exit_time_msc
```

stores the latest supported exit execution time in milliseconds.

The value is derived from:

```text
DEAL_TIME_MSC
```

for supported `DEAL_ENTRY_OUT` Deals.

The value does not independently imply lifecycle completion.

---

## 5. Derived Values

The following lifecycle values are derived from stored state and must not be stored independently:

```text
remaining_volume
entry_price
exit_price
is_complete
```

Derived values must be calculated deterministically from accumulated evidence.

This prevents duplicated state from becoming inconsistent.

---

## 6. Remaining Volume

Remaining executed exposure is derived as:

```text
remaining_volume = entry_volume - exit_volume
```

Volume precision must be handled according to the applicable MQL5 symbol volume precision.

The lifecycle state must not preserve a separate mutable `remaining_volume` field.

For example, the following contradictory state must be impossible by construction:

```text
entry_volume     = 0.10
exit_volume      = 0.10
remaining_volume = 0.05
```

---

## 7. Entry Price

The reconstructed entry price is derived from entry execution evidence.

If:

```text
entry_volume > 0
```

then:

```text
entry_price = entry_value / entry_volume
```

If no supported entry execution has been accumulated, the lifecycle does not yet expose a valid reconstructed entry price.

The exact MQL5 API representation of an unavailable derived value is an implementation detail.

---

## 8. Exit Price

The reconstructed exit price is derived from exit execution evidence.

If:

```text
exit_volume > 0
```

then:

```text
exit_price = exit_value / exit_volume
```

If no supported exit execution has been accumulated, the lifecycle does not yet expose a valid reconstructed exit price.

The exact MQL5 API representation of an unavailable derived value is an implementation detail.

---

## 9. Lifecycle Completion

Lifecycle completion is derived from accumulated execution volume.

Conceptually, a lifecycle is complete when:

```text
entry_volume > 0
```

and:

```text
remaining_volume == 0
```

within the applicable volume precision.

The lifecycle state must not preserve a mutable `is_complete` field.

Completion must always be derived from execution evidence.

This prevents contradictory state such as:

```text
entry_volume = 0.10
exit_volume  = 0.10
is_complete  = false
```

---

## 10. Direction Model

The lifecycle state uses an internal direction abstraction rather than storing `ENUM_POSITION_TYPE` directly.

Conceptually:

```text
MQL5_LIFECYCLE_DIRECTION_UNKNOWN
MQL5_LIFECYCLE_DIRECTION_LONG
MQL5_LIFECYCLE_DIRECTION_SHORT
```

The exact enum declaration belongs to the MQL5 implementation.

The direction abstraction exists because lifecycle direction is reconstructed from entry execution evidence rather than copied from current Position state.

This also avoids making current Position availability a prerequisite for lifecycle reconstruction.

---

## 11. State Initialization

A lifecycle state is created when the reconstruction component encounters the first supported Deal for a previously unseen `DEAL_POSITION_ID`.

The state is initialized with:

```text
position_id = DEAL_POSITION_ID
```

Accumulated numeric execution fields begin at zero.

Conceptually:

```text
entry_volume       = 0
exit_volume        = 0
entry_value        = 0
exit_value         = 0
commission         = 0
swap               = 0
realized_profit    = 0
first_entry_time_msc = unset
last_exit_time_msc   = unset
direction            = UNKNOWN
```

Stable lifecycle metadata is established from supported entry execution evidence.

---

## 12. Entry Evidence Application

When supported entry execution evidence is applied to the lifecycle state, the state updates:

```text
entry_volume
entry_value
commission
swap
realized_profit
first_entry_time_msc
```

The first supported entry execution also establishes:

```text
symbol
magic
direction
```

Subsequent supported entry evidence must be consistent with established lifecycle metadata.

A metadata inconsistency must not silently overwrite the existing lifecycle state.

The handling of inconsistent evidence belongs to the reconstruction component.

---

## 13. Exit Evidence Application

When supported exit execution evidence is applied to the lifecycle state, the state updates:

```text
exit_volume
exit_value
commission
swap
realized_profit
last_exit_time_msc
```

Exit evidence must not redefine:

```text
symbol
magic
direction
```

Lifecycle direction remains the direction established by supported entry execution evidence.

---

## 14. Duplicate Deal Tracking

`Mql5LifecycleState` does not track processed Deal tickets.

The state object assumes that execution evidence passed to it has already passed duplicate protection.

Conceptually:

```text
Mql5LifecycleReconstructor
        |
        +-- tracks processed DEAL_TICKET values
        +-- correlates DEAL_POSITION_ID values
        +-- validates supported evidence
        |
        v
Mql5LifecycleState
        |
        +-- accumulates lifecycle facts
        +-- exposes derived lifecycle values
```

Duplicate Deal protection is a reconstruction responsibility, not lifecycle state.

This separation prevents the state object from becoming an ingestion coordinator.

---

## 15. Current Position State

`Mql5LifecycleState` does not require current MetaTrader 5 Position state.

It must not depend on:

```text
PositionSelectByTicket
POSITION_VOLUME
POSITION_PRICE_OPEN
POSITION_SL
POSITION_TP
POSITION_PROFIT
```

for execution reconstruction.

Current Position state may be observed elsewhere by the Adapter for optional live enrichment.

It is not part of the minimal lifecycle state defined by this document.

---

## 16. Operational Mutations

The lifecycle state does not preserve operational mutation history.

The initial state model therefore does not store:

```text
stop_loss
take_profit
stop_loss_history
take_profit_history
```

Operations such as:

```text
TRADE_ACTION_SLTP
```

do not modify lifecycle execution accumulation.

A future operational-state capability may model such mutations separately.

That capability must not be mixed into the initial Deal-based lifecycle state.

---

## 17. Non-Responsibilities

`Mql5LifecycleState` is not responsible for:

- receiving `OnTradeTransaction()` callbacks;
- loading Deal history from MetaTrader 5;
- detecting duplicate Deal tickets;
- finding or creating lifecycle instances;
- deciding whether a `DEAL_ENTRY` value is supported;
- handling unsupported platform evidence;
- inferring strategy intent;
- classifying scale-in or scale-out decisions;
- observing current Position state;
- constructing canonical `TradeRecord` values;
- persisting lifecycle state;
- calculating trading statistics.

These responsibilities belong to reconstruction, mapping, storage, or downstream research components.

---

## 18. State Invariants

The lifecycle state must preserve the following invariants.

### I1 — Position ID Is Immutable

The lifecycle `position_id` must not change after construction.

### I2 — Lifecycle Metadata Is Not Correlation State

`symbol` and `magic` are preserved as metadata but must not identify the lifecycle.

### I3 — Direction Comes from Entry Evidence

Lifecycle direction is established only from supported entry execution evidence.

Exit Deal type must not redefine lifecycle direction.

### I4 — Entry Volume Is Accumulated

`entry_volume` contains only accumulated supported entry execution volume.

### I5 — Exit Volume Is Accumulated

`exit_volume` contains only accumulated supported exit execution volume.

### I6 — Prices Are Derived

Reconstructed entry and exit prices are derived from accumulated execution value and volume.

### I7 — Remaining Volume Is Derived

Remaining volume is never stored independently.

### I8 — Completion Is Derived

Lifecycle completion is never stored independently.

### I9 — Current Position State Is Not Required

The lifecycle remains reconstructable after the corresponding current Position disappears.

### I10 — Strategy Intent Is Absent

The state contains platform lifecycle facts and does not encode strategy intent.

---

## 19. Preliminary MQL5 Shape

The following shape illustrates the intended implementation boundary:

```mql5
class Mql5LifecycleState
{
private:
   ulong m_position_id;
   string m_symbol;
   long m_magic;

   Mql5LifecycleDirection m_direction;

   double m_entry_volume;
   double m_exit_volume;

   double m_entry_value;
   double m_exit_value;

   double m_commission;
   double m_swap;
   double m_realized_profit;

   long m_first_entry_time_msc;
   long m_last_exit_time_msc;

public:
   // construction
   // supported evidence application
   // stored-state accessors
   // deterministic derived-value accessors
};
```

This is a design shape, not the final implementation contract.

Method names and exact MQL5 representation details should be defined during implementation.

The implementation should remain minimal and preserve the responsibilities and invariants defined by this document.

---

## 20. Relationship to the Reconstructor

The lifecycle state is intentionally passive with respect to platform ingestion.

The future reconstruction component is responsible for orchestration.

Conceptually:

```text
OnTradeTransaction
        |
        v
Mql5LifecycleReconstructor
        |
        +-- validate transaction type
        +-- load Deal evidence
        +-- reject duplicates
        +-- classify Deal entry type
        +-- correlate DEAL_POSITION_ID
        +-- find or create lifecycle state
        |
        v
Mql5LifecycleState
        |
        +-- apply entry evidence
        +-- apply exit evidence
        +-- expose reconstructed facts
```

This boundary keeps platform event handling separate from lifecycle state accumulation.

---

## 21. Conclusion

`Mql5LifecycleState` is the minimal internal state required to accumulate supported MQL5 Deal evidence for one reconstructed Position lifecycle.

It stores only stable metadata and accumulated execution facts.

Remaining volume, reconstructed prices, and lifecycle completion are derived deterministically rather than stored as mutable duplicate state.

The state does not track Deal ingestion, current Position state, operational mutations, or strategy intent.

This boundary prepares the MQL5 Adapter for a separate `Mql5LifecycleReconstructor` component responsible for Deal ingestion, duplicate protection, lifecycle correlation, and unsupported-evidence handling.