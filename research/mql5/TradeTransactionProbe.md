

# Trade Transaction Probe

## 1. Purpose

This research note documents an empirical investigation into the trade transaction lifecycle exposed by MetaTrader 5 through `OnTradeTransaction()`.

The purpose of the experiment is not to define canonical EdgeLab DAF semantics directly.

Instead, the probe observes platform-level facts and records how MetaTrader 5 represents a small set of common trading operations.

The resulting evidence is used to inform the design of the MQL5 Platform Adapter and its lifecycle reconstruction rules.

The probe intentionally avoids domain-level interpretation wherever possible.

---

## 2. Research Question

The experiment investigates the following question:

> How does MetaTrader 5 represent the lifecycle of a position across opening, operational modification, partial exit, same-direction entry, and final exit?

A secondary goal is to identify which MQL5 identifiers and transaction properties may be suitable for reconstructing a stable trade lifecycle.

---

## 3. Experimental Environment

The probe was executed in the MetaTrader 5 Strategy Tester.

Observed environment:

| Property | Value |
|---|---|
| Platform | MetaTrader 5 |
| Symbol | `EURUSD+` |
| Timeframe | `M1` |
| Test date | `2025-12-15` |
| Account margin mode | `ACCOUNT_MARGIN_MODE_RETAIL_HEDGING` |
| Probe magic number | `26062801` |
| Initial volume | `0.10` |
| Partial-close volume | `0.05` |
| Same-direction entry volume | `0.05` |

The observations in this document apply specifically to the tested hedging accounting model.

They must not be assumed to describe netting-account behavior without separate empirical verification.

---

## 4. Probe Design

The diagnostic Expert Advisor executes one scenario per new bar.

This separation makes the resulting transaction groups easier to inspect in the Strategy Tester journal.

The probe executes the following scenarios:

| Scenario | Operation |
|---|---|
| `S01` | Open Position |
| `S02` | Modify Stop Loss |
| `S03` | Partial Close |
| `S04` | Same-Direction Entry |
| `S05` | Full Close |

For every invocation of `OnTradeTransaction()`, the probe records:

- `MqlTradeTransaction` fields;
- `MqlTradeRequest` and `MqlTradeResult` for request transactions;
- Deal history data for `TRADE_TRANSACTION_DEAL_ADD`;
- current Position state when the referenced Position is still selectable.

The probe is diagnostic research code. It is not part of the EdgeLab DAF runtime implementation.

---

## 5. Scenario S01 — Open Position

### Operation

The probe opens a market BUY Position with volume `0.10`.

### Observed Deal

The execution produced a Deal with the following relevant properties:

```text
DEAL_TYPE_BUY
DEAL_ENTRY_IN
DEAL_POSITION_ID = 2
volume = 0.10
magic = 26062801
reason = DEAL_REASON_EXPERT
```

The resulting Position exposed:

```text
POSITION_TICKET     = 2
POSITION_IDENTIFIER = 2
volume              = 0.10
```

### Observed Transaction Sequence

The callback sequence recorded by the probe was:

```text
TRADE_TRANSACTION_DEAL_ADD
TRADE_TRANSACTION_ORDER_DELETE
TRADE_TRANSACTION_HISTORY_ADD
TRADE_TRANSACTION_REQUEST
```

The request callback was observed after the Deal and order-history related callbacks.

### Observation

A single trading operation produced multiple transaction callbacks.

The observed callback order did not follow a simple conceptual sequence such as:

```text
REQUEST -> ORDER -> DEAL -> POSITION
```

### Initial Implication

The MQL5 Adapter must not assume that one `OnTradeTransaction()` invocation represents one complete trade event.

It must also avoid depending on an assumed callback ordering when reconstructing lifecycle state.

---

## 6. Scenario S02 — Modify Stop Loss

### Operation

The probe modifies the Stop Loss of Position `2`.

The Stop Loss changes from:

```text
0.00000
```

to:

```text
1.17211
```

### Observed Transaction

The operation produced:

```text
TRADE_TRANSACTION_REQUEST
```

with:

```text
action   = TRADE_ACTION_SLTP
position = 2
sl       = 1.17211
```

No `TRADE_TRANSACTION_DEAL_ADD` was observed for the Stop Loss modification.

### Observation

The operation changed the current operational state of the Position but did not create an execution Deal.

The underlying execution facts remained unchanged:

```text
POSITION_IDENTIFIER = 2
volume              = 0.10
price_open          = 1.17434
```

### Initial Implication

Stop Loss and Take Profit changes must not be interpreted as trade executions.

This suggests a conceptual separation between:

```text
Execution facts
---------------
entry executions
exit executions
executed volume
execution price
costs

Operational position state
--------------------------
current volume
stop loss
take profit
current market price
```

Operational state may change over time without changing the historical execution facts of the trade lifecycle.

---

## 7. Scenario S03 — Partial Close

### Operation

The probe closes `0.05` of the existing `0.10` BUY Position.

### Observed Deal

The closing execution produced:

```text
DEAL_TYPE_SELL
DEAL_ENTRY_OUT
DEAL_POSITION_ID = 2
volume = 0.05
profit = -6.15
```

### Observed Position State

After the Deal, Position `2` was still available:

```text
POSITION_IDENTIFIER = 2
volume              = 0.05
```

### Observation

`DEAL_ENTRY_OUT` represents exit execution evidence, but it does not independently indicate that the Position lifecycle has completed.

In this scenario:

```text
DEAL_ENTRY_OUT
```

was followed by a still-existing Position with remaining volume.

### Initial Implication

The following interpretation is invalid:

```text
DEAL_ENTRY_OUT -> Trade Closed
```

The Adapter must distinguish between partial and final exits through lifecycle reconstruction rather than through `DEAL_ENTRY_OUT` alone.

Conceptually:

```text
DEAL_ENTRY_OUT
      |
      v
Does lifecycle exposure remain?
      |
  +---+---+
  |       |
 YES      NO
  |       |
  v       v
Partial   Final
Exit      Exit
```

The exact reconstruction mechanism remains a design decision to be derived from the available execution history.

---

## 8. Scenario S04 — Same-Direction Entry

### Operation

While Position `2` remained open as a BUY Position with volume `0.05`, the probe submitted another BUY market operation for volume `0.05` on the same symbol.

### Observed Deal

The new execution produced:

```text
DEAL_TYPE_BUY
DEAL_ENTRY_IN
DEAL_POSITION_ID = 4
volume = 0.05
```

### Observed Position State

The resulting Position exposed:

```text
POSITION_TICKET     = 4
POSITION_IDENTIFIER = 4
volume              = 0.05
```

The original Position retained:

```text
POSITION_IDENTIFIER = 2
```

### Observation

Under the observed hedging accounting model, a same-direction entry did not increase the volume of Position `2`.

Instead, MetaTrader 5 created a new independent Position lifecycle:

```text
Position 2
BUY 0.05

Position 4
BUY 0.05
```

### Initial Implication

A same-direction execution must not automatically be interpreted as a scale-in operation by the Platform Adapter.

`Scale In` is strategy-level intent.

The observed platform fact is:

```text
New DEAL_ENTRY_IN
New DEAL_POSITION_ID
New POSITION_IDENTIFIER
```

The Adapter must preserve platform evidence without inferring strategy intent that is not explicitly represented by the platform data.

The scenario also demonstrates that neither symbol nor `symbol + magic` is sufficient to identify an independent Position lifecycle on a hedging account.

---

## 9. Scenario S05 — Full Close

### Operation

The probe closes the remaining `0.05` volume of Position `2`.

### Observed Deal

The final execution produced:

```text
DEAL_TYPE_SELL
DEAL_ENTRY_OUT
DEAL_POSITION_ID = 2
volume = 0.05
profit = -6.15
```

### Observed Position Availability

During transaction processing, the probe attempted to select Position `2` and recorded:

```text
Position not available: 2
```

The Position was no longer available through the current Position state.

### Observation

A final exit Deal may be observable after the associated Position has already disappeared from the current Position set.

### Initial Implication

Historical lifecycle reconstruction must not require the current Position to remain selectable.

An Adapter implementation based exclusively on logic such as:

```mql5
if(PositionSelectByTicket(trans.position))
{
   // reconstruct lifecycle
}
```

would be fragile for completed Position lifecycles.

Historical execution evidence must be sufficient to reconstruct completed lifecycles independently of current Position availability.

---

## 10. Reconstructed Observed Lifecycles

The experiment produced two independent Position lifecycles.

### Position Lifecycle 2

```text
DEAL #2
DEAL_ENTRY_IN
volume = 0.10
position_id = 2

        |
        v

DEAL #3
DEAL_ENTRY_OUT
volume = 0.05
position_id = 2

        |
        v

DEAL #5
DEAL_ENTRY_OUT
volume = 0.05
position_id = 2

        |
        v

Position no longer available
```

Execution-volume reconstruction:

```text
Entry volume = 0.10
Exit volume  = 0.05 + 0.05
             = 0.10
```

Observed lifecycle result:

```text
0.10 entered
0.10 exited
lifecycle completed
```

### Position Lifecycle 4

```text
DEAL #4
DEAL_ENTRY_IN
volume = 0.05
position_id = 4
```

The Position remained open after scenario `S05` because it was independent from Position `2`.

At the end of the Strategy Tester run, the tester closed Position `4` automatically:

```text
position closed due end of test
```

This confirms that scenario `S04` created an independent Position rather than modifying the lifecycle identified by `DEAL_POSITION_ID = 2`.

---

## 11. Evidence and Adapter Implications

| Observed Evidence | Adapter Implication |
|---|---|
| One trading operation produced multiple transaction callbacks | Do not map one callback directly to one complete trade event |
| The request callback was observed after Deal-related callbacks | Do not rely on callback arrival order |
| `TRADE_ACTION_SLTP` produced no Deal | Operational Position mutations are not executions |
| `DEAL_ENTRY_OUT` left the Position alive after a partial close | `DEAL_ENTRY_OUT` alone does not prove lifecycle completion |
| `DEAL_POSITION_ID = 2` remained stable across entry and exit Deals | `DEAL_POSITION_ID` is a strong lifecycle correlation candidate |
| Same-direction entry created `DEAL_POSITION_ID = 4` on the hedging account | Do not infer scale-in semantics from direction and symbol alone |
| Two Positions shared symbol and magic number | Symbol and magic number are metadata, not lifecycle identifiers |
| The fully closed Position was no longer selectable during callback processing | Historical reconstruction must not depend on current Position availability |

---

## 12. Preliminary MQL5 Reconstruction Rules

The following rules are preliminary and derived only from the observed hedging-account experiment.

They are research conclusions, not yet stable public framework guarantees.

### R1 — Deal Add Is Primary Execution Evidence

`TRADE_TRANSACTION_DEAL_ADD` provides the primary observable evidence that an execution occurred.

### R2 — Deal Position ID Is a Lifecycle Correlation Candidate

`DEAL_POSITION_ID` remained stable across the observed entry, partial exit, and final exit Deals belonging to Position lifecycle `2`.

It is therefore a strong candidate for MQL5 lifecycle correlation.

### R3 — Entry Deals Contribute Entry Execution Volume

A Deal with:

```text
DEAL_ENTRY_IN
```

contributes executed entry volume to the correlated lifecycle.

### R4 — Exit Deals Contribute Exit Execution Volume

A Deal with:

```text
DEAL_ENTRY_OUT
```

contributes executed exit volume to the correlated lifecycle.

### R5 — Exit Entry Type Does Not Prove Lifecycle Completion

`DEAL_ENTRY_OUT` alone must not be interpreted as lifecycle completion.

Lifecycle completion requires reconstruction from the correlated execution evidence.

### R6 — Current Position State Is Optional Reconstruction Evidence

Current Position state may enrich live observation but must not be required to reconstruct a historical lifecycle.

### R7 — SL/TP Mutation Is Operational State Mutation

`TRADE_ACTION_SLTP` represents a change to operational Position state and must not be treated as execution evidence.

### R8 — Symbol and Magic Are Not Lifecycle Identifiers

Symbol and magic number may be retained as metadata and filtering attributes.

They must not be used as the primary key for correlating independent MQL5 Position lifecycles.

---

## 13. Open Questions

The experiment intentionally leaves several questions unresolved.

### 13.1 Netting Accounts

The observed environment used:

```text
ACCOUNT_MARGIN_MODE_RETAIL_HEDGING
```

Equivalent scenarios have not yet been empirically observed under a netting accounting model.

No netting-specific Adapter behavior should be implemented solely from assumptions.

### 13.2 Reversal Deals

The probe did not investigate:

```text
DEAL_ENTRY_INOUT
```

Reversal behavior requires a dedicated experiment.

### 13.3 Close-By Operations

The probe did not investigate:

```text
DEAL_ENTRY_OUT_BY
```

Close-by semantics require separate observation.

### 13.4 Pending Orders and Partial Fills

The experiment used market operations and did not investigate:

- pending-order activation;
- rejected requests;
- cancelled orders;
- partial fills;
- multiple Deals generated from one order.

These cases may affect transaction correlation and Adapter buffering requirements.

### 13.5 Platform-Generated Exits

The experiment did not intentionally trigger exits caused by:

- Stop Loss;
- Take Profit;
- Stop Out.

`DEAL_REASON` should be investigated before defining canonical exit-reason semantics.

---

## 14. Conclusion

The experiment demonstrates that MetaTrader 5 trade lifecycle reconstruction cannot be based on a one-callback-to-one-event mapping.

The observed hedging-account behavior shows that:

- execution evidence is represented through Deals;
- operational Position mutations may occur without Deals;
- exit Deals may represent partial or final exits;
- completed Positions may already be absent from current Position state during callback processing;
- same-direction entries may create independent Position lifecycles;
- symbol and magic number are insufficient lifecycle correlation keys;
- `DEAL_POSITION_ID` is a strong candidate for correlating execution history into MQL5 Position lifecycles.

These observations provide the first empirical basis for the MQL5 Platform Adapter reconstruction model.

The next design step is to translate the observed evidence into a minimal Adapter reconstruction specification before implementing the first runtime classes.