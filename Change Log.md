Change Log

========================================================================
Version: 1.0 (Initial Request)
========================================================================

- Base Strategy:
  - Entry: Price relative to EMA 9, 20, 50. Distance check from EMA 9. Volume > Volume MA confirmation.
  - Grid: Open additional trades at fixed point distance (`DistancePoints`). Lot size increases by fixed `StepLot`.
  - Exit: Close all positions of a direction when price hits EMA 20.
- Inputs: `Lot`, `StepLot`, `DistancePoints`.

========================================================================
Version: 1.1 (Compiler Fix)
========================================================================

- Fixed MQL4 `iMAOnArray` usage error.
- Implemented MQL5 method for calculating Moving Average on Volume data using indicator handles (`iVolumes`, `iMA` applied to handle).

========================================================================
Version: 1.2 (Compiler Fix)
========================================================================

- Fixed MQL5 `SYMBOL_VOLUME_DIGITS` undeclared identifier error.
- Introduced `VolumeDigits` helper function to calculate lot digits based on `SYMBOL_VOLUME_STEP`.

========================================================================
Version: 1.3 (Volume Type Change)
========================================================================

- Changed the volume indicator (`iVolumes`) to prioritize using `VOLUME_TICK` (Tick Volume) with a fallback to `VOLUME_REAL` (Real Volume).

========================================================================
Version: 1.4 (ATR Distance & Delayed Exit)
========================================================================

- Replaced fixed `DistancePoints` input with dynamic calculation: ATR(AtrPeriod) \* AtrCoefficient.
- Modified exit logic: If price hits EMA 20 but total floating equity for that direction is negative, delay closure and wait for price to hit EMA 50 instead.
- Added `AtrPeriod`, `AtrCoefficient` inputs.
- Added `atrHandle` global variable and ATR initialization/release.
- Added `GetAtrDistancePoints` (later `GetAtrDistancePrice`) function.
- Added `delayBuyExit`, `delaySellExit` flags.
- Added `CalculateFloatingEquity` function.

========================================================================
Version: 1.5 (Time/Spread Filters)
========================================================================

- Added optional Time Filter: Prevent new initial entries outside specified start/end times (server time).
  - Inputs: `EnableTimeFilter`, `TradingStartTime`, `TradingEndTime`.
  - Helper functions: `TimeToMinutes`, `IsTradingAllowedTime`.
- Added optional Spread Filter: Prevent new initial _and_ grid entries if current spread > `MaxSpreadPoints`.
  - Input: `MaxSpreadPoints`.
  - Helper function: `IsSpreadOk`.
- Entry and Grid logic modified to check these filters.

========================================================================
Version: 1.6 (Time Filter Fix)
========================================================================

- Fixed "constant cannot be modified" error related to `EnableTimeFilter`.
- Introduced internal `timeFilterActive` flag to manage operational status based on user input and successful time parsing in `OnInit`.

========================================================================
Version: 1.7 (ATR Trailing Stop)
========================================================================

- Replaced EMA 20 / EMA 50 direct exit logic.
- Implemented ATR-based Trailing Stop:
  - Trigger: When EMA 20 (or EMA 50 if delayed due to negative equity) is hit.
  - Initial Stop: Placed ATR\*Coefficient distance away from the trigger price.
  - Trailing: Stop Loss is moved in steps of ATR\*Coefficient as price moves favorably.
- Added trailing state variables (`isBuyTrailingActive`, `buyTrailStopPrice`, etc.).
- Added `ManageTrailingStop` function.
- Added `ModifyAllPositionsSL` helper function.
- Modified `CheckExitConditions` to trigger trailing instead of closing.
- Added logic to reset trailing state when positions are closed (`CheckAndResetTrailingState`).

========================================================================
Version: 1.8 (Break-Even Trail Step)
========================================================================

- Modified ATR Trailing Stop logic (`ManageTrailingStop`).
- Trailing Stop Loss will now only move _further_ into profit if the new potential SL level is at or better than the calculated break-even price of all open positions for that direction.
- Added `CalculateAverageEntryPrice` helper function.

========================================================================
Version: 1.9 - 1.9.5 (Equity Trail Attempt & Debugging)
========================================================================

- Replaced ATR Trailing Stop with an Equity-Based Trailing Stop.
  - Inputs: `EnableEquityTrailing`, `EquityTrailStart`, `EquityTrailStep`.
  - Logic: Activate at `EquityTrailStart`, lock `Start - Step`, move stop up by `Step` for every `Step` increase in highest equity. Close if equity drops below `equityTrailStopLevel`.
- Removed ATR Trailing functions and variables.
- Reverted distance calculation to fixed `DistancePoints` input temporarily.
- Multiple debugging versions (1.9.1 - 1.9.5) added logging and fixed input validation/state reset issues related to the Equity Trail logic, addressing reports that it wasn't trailing or closing correctly.

========================================================================
Version: 2.0 - 2.1.2 (Simplified Equity TP & News Filter Attempt)
========================================================================

- Removed complex Equity Trailing Stop logic.
- Implemented simple Equity Take Profit: Close all positions when total floating equity >= `EquityTakeProfit` input ($10 default).
- Removed Equity Trailing inputs/globals/functions.
- Attempted to add a News Filter using built-in Calendar functions (v2.1).
  - Inputs: `EnableNewsFilter`, `NewsFilterMinutesBefore/After`, `NewsFilterImportance`.
  - Function: `IsNewsTime`.
- Encountered and fixed compiler errors related to incorrect MQL5 identifiers (`SYMBOL_CURRENCY_QUOTE`, `IMPORTANCE_HIGH`, `importance_id`) and potentially `CalendarValues` recognition (v2.1.1, v2.1.2).
- **Note:** The News Filter remained problematic due to persistent compiler errors related to `CalendarValues`, likely indicating an environment issue on the user's side.

========================================================================
Version: 2.2 - 2.2.1 (Bypass Filters / Removed News Filter)
========================================================================

- **Removed News Filter** due to persistent compilation issues.
- Removed general Time Filter inputs/logic (`EnableTimeFilter`, `TradingStartTime`, `TradingEndTime`, `IsTradingAllowedTime`).
- Added specific **Hourly Bypass Filter**: Don't open _initial_ trades between `BypassStartHour` and `BypassEndHour`.
  - Inputs: `BypassHourlyEnable`, `BypassStartHour`, `BypassEndHour`.
  - Function: `IsHourBypassed`.
- Added specific **Daily Bypass Filter**: Don't open _initial_ trades on Friday.
  - Input: `BypassFridayEnable`.
  - Function: `IsDayBypassed`.
- Reverted distance logic back to fixed `DistancePoints` input.
- Fixed compiler errors from leftover general time filter variable references (v2.2.1).

========================================================================
Version: 2.3 (Reinstated ATR Distance)
========================================================================

- Removed fixed `DistancePoints` input.
- Re-added `AtrPeriod`, `AtrCoefficient` inputs.
- Re-added `atrHandle` global and initialization/release.
- Re-added `GetAtrDistancePrice` function.
- Modified `CheckEntryConditions` and `ManageGrid` to use the dynamic ATR-based distance calculation again.
- Kept simple Equity TP exit and specific Hourly/Daily bypass filters.

========================================================================
Version: 2.4 (Added EMA 200 Filter - Initial Incorrect Logic)
========================================================================

- Added `emaHandle200` global variable, initialization, release.
- Modified `CheckEntryConditions` to include EMA 200 check.
- **Initial (Incorrect) Logic:** Buy if Price > EMA 200, Sell if Price < EMA 200 (Trend Filter).

========================================================================
Version: 2.5 - 2.5.1 (Revised Waterfall Exit & Array Fix)
========================================================================

- Replaced simple Equity Take Profit with a new conditional **EMA Waterfall Exit Logic** (activated only if `EquityTakeProfit <= 0`).
  - Logic: EMA9 hit closes if equity >= `WaterfallMinProfit`, else waits for EMA20. EMA20 hit (if waiting) closes if equity >= `WaterfallMinProfit`, else waits for EMA50. EMA50 hit (if waiting) closes if equity >= `WaterfallMinProfit`, else waits for EMA200. EMA200 hit (if waiting) closes regardless of profit.
  - Added `WaterfallMinProfit` input.
  - Added `waitFor...` state flags.
  - Created `CheckExitConditions` function to handle both Equity TP and Waterfall logic.
- Fixed "array out of range" bug in `GetIndicatorValue` helper function by adding `ArrayResize` (v2.5.1).

========================================================================
Version: 2.6 - 2.6.1 (Market Closed Fix & Corrected EMA 200 Filter)
========================================================================

- Added `IsTradingEnabledNow` function to check `TERMINAL_TRADE_ALLOWED`, `TERMINAL_CONNECTED`, and `SYMBOL_TRADE_MODE`.
- Added checks using `IsTradingEnabledNow` before attempting position closures in `CheckExitConditions` and `CloseAllPositionsOfType` to prevent "Market Closed" errors stopping execution (though retry logic wasn't fully implemented yet). Also added check before Entries and Grid management.
- **Corrected EMA 200 Filter Logic** in `CheckEntryConditions`:
  - Buy requires `closePrice < ema200` (along with < E9/20/50).
  - Sell requires `closePrice > ema200` (along with > E9/20/50).
- Fixed "undeclared identifier" for `IsTradingEnabledNow` by moving its definition earlier in the file (v2.6.1).

========================================================================
Version: 2.7 - 2.7.1 (Grid Lot Coefficient)
========================================================================

- Replaced `StepLot` input with `LotCoefficient` input.
- Changed grid lot calculation from additive to multiplicative: `NewLot = LastLot * LotCoefficient`.
- Added `lastBuyLot`, `lastSellLot` global variables to track last used lot size.
- Modified `CheckEntryConditions`, `ManageGrid`, `CheckAndResetGridState`, `CloseAllPositionsOfType` to store, use, and reset these new lot variables.
- Fixed "undeclared identifier" errors for `LotCoefficient`, `lastBuyLot`, `lastSellLot` (v2.7.1).

========================================================================
Version: 2.8 - 2.8.1 (Dynamic Equity TP)
========================================================================

- Modified Equity Take Profit logic (when `EquityTakeProfit > 0`):
  - Target is now calculated as: `dynamicEquityTarget = EquityTakeProfit * NumberOfOpenOrders`.
  - The `EquityTakeProfit` input now represents the target profit _per open order_.
- Added `PositionsTotalFiltered` helper function to count only EA's positions.
- Modified `CheckExitConditions` to implement this dynamic calculation.
- Added logging to verify the dynamic calculation (v2.8.1).

========================================================================
Version: 3.0 - 3.0.4 (Revised Waterfall + Market Closed Retry + Fixes)
========================================================================

- Revised EMA Waterfall logic again based on user clarification (Check profit >= `WaterfallMinProfit` at E9/20/50 hits).
- Re-introduced pending closure flags (`pendingEquityTP_Close`, `pendingWaterfallBuy_Close`, `pendingWaterfallSell_Close`) and `ResetPendingCloseFlags` function.
- Re-introduced `HandlePendingClosures` function to retry closing positions when the market reopens after a "Market Closed" error.
- Refined `CheckExitConditions` to set pending flags when market is closed instead of attempting to close.
- Refined `CloseAllPositionsOfType` to ONLY attempt closing, moving state resets primarily to `CheckAndResetGridState` or the calling function (`HandlePendingClosures`, `CheckExitConditions`).
- Fixed various compiler errors related to parameter counts (`CheckExitConditions`) and unbalanced parentheses (`OnInit`) through code restructuring and careful checking (v3.0.1 - v3.0.4).

========================================================================
Version: 3.1 - 3.1.2 (Final Waterfall + Retry Refinement + Fixes)
========================================================================

- Implemented final user-specified EMA Waterfall logic with `WaterfallMinProfit` check at EMA9, EMA20, EMA50, and unconditional close at EMA200 if waiting.
- Refined `HandlePendingClosures` retry logic and flag resetting based on remaining positions _after_ closure attempts.
- Refined `CheckAndResetGridState` to also reset pending flags when positions disappear.
- Refined `CloseAllPositionsOfType` to remove state resets entirely, deferring them to `CheckAndResetGridState` or the calling function logic.
- Fixed final `'unbalanced parentheses'` error pointing at `OnInit` by reformatting/simplifying the indicator initialization block within `OnInit` (v3.1.2).

========================================================================

//--------------------------------------------------------------------
// Version 3.4.4 (Current - Based on last request)
// Date: 2025-05-09
//--------------------------------------------------------------------
// - Feature: Individual Hedging for Subsequent Counter Grid Orders.
//   - If a Counter Grid (Buy or Sell) has already placed its initial overall hedge
//     (i.e., `isCounterBuyGridSelfHedged` or `isCounterSellGridSelfHedged` is true),
//     any NEW grid orders added to THAT Counter Grid will now also be individually
//     hedged with an opposing order of the same lot size, using `counterGridHedgeMagicNumber`.
//   - Added new function `OpenIndividualCounterOrderHedge()` to handle this.
// - Feature: Conditional Equity Trailing P/L Source.
//   - Modified `CheckExitConditions()`:
//     - If `isCounterBuyGridSelfHedged` OR `isCounterSellGridSelfHedged` is true,
//       the `currentFloatingEquity` for the Equity Trailing Stop is calculated
//       based ONLY on the Profit/Loss of the Main Grid (`MagicNumber`).
//     - Otherwise (if neither counter grid has self-hedged yet), the
//       `currentFloatingEquity` is based on the combined P/L of the
//       Main Grid (`MagicNumber`) AND the Counter Grid (`counterMagicNumber`).
//     - P/L from orders with `counterGridHedgeMagicNumber` (hedges for the counter grid)
//       is NOT included in the `currentFloatingEquity` for trailing stop decisions.
// - State Management:
//   - Added `isCounterBuyGridSelfHedged` and `isCounterSellGridSelfHedged` global boolean flags.
//   - These flags are initialized in `OnInit()`, saved/loaded via `SaveTrailingState()`/`LoadTrailingState()`.
//   - These flags are set in `OpenHedgeForCounterGrid()` when the initial overall hedge is placed.
//   - These flags are reset in `CheckAndResetGridState()` when the corresponding counter grid is fully closed.
// - `ManageGrid()`: Logic updated to call `OpenIndividualCounterOrderHedge()` if the parent Counter Grid is already self-hedged,
//   or call `OpenHedgeForCounterGrid()` if the Counter Grid reaches `MaxGridLevelBeforeClose` for the first time.

//--------------------------------------------------------------------
// Version 3.4.3
//--------------------------------------------------------------------
// - Feature: Added Hedging for the Counter Grid when it reaches MaxGridLevelBeforeClose.
//   - When a Counter Grid (Buy or Sell) independently reaches `MaxGridLevelBeforeClose`,
//     it will place a single hedge order (opposite direction) for the *entire sum of lots*
//     of that Counter Grid. This hedge uses a new `counterGridHedgeMagicNumber`.
//   - This action does NOT close any other positions or stop the Main Grid.
// - New Globals: `counterGridHedgeMagicNumber` and `CTrade counterGridHedgeTrade`.
// - New Function: `OpenHedgeForCounterGrid()` to implement the above hedging logic.
// - `ManageGrid()`: Updated to call `OpenHedgeForCounterGrid()` when a counter grid
//   (Counter Buy or Counter Sell) reaches `MaxGridLevelBeforeClose`.
// - `CloseAllPositionsOfType()`: Updated to handle closing orders with `counterGridHedgeMagicNumber`.
// - `HandlePendingClosures()` & `CheckExitConditions()`: Updated to ensure positions with
//   `counterGridHedgeMagicNumber` are closed during system-wide closures.
// - `CheckAndResetGridState()`: Updated to consider `counterGridHedgeMagicNumber` positions
//   when determining if all positions are closed for resetting `isEquityTrailingActive`.

//--------------------------------------------------------------------
// Version 3.4.2
//--------------------------------------------------------------------
// - Feature: `MaxGridLevelBeforeClose` now also applies to the Counter Grid's independent growth.
//   - If the Counter Buy Grid OR Counter Sell Grid reaches `MaxGridLevelBeforeClose`
//     due to its own grid additions, ALL positions (Main, Counter) will be closed.
// - `ManageGrid()`: Added checks for `counterBuyGridCount` and `counterSellGridCount`
//   against `MaxGridLevelBeforeClose` in the sections managing independent counter grid growth.
//   If met, it triggers closure of all Main and Counter grid positions.

//--------------------------------------------------------------------
// Version 3.4.1
//--------------------------------------------------------------------
// - Feature: Equity Trailing Stop now considers the combined P/L of Main Grid AND Counter Grid.
// - `CheckExitConditions()`: Modified the calculation of `currentFloatingEquity` to sum
//   the P/L from orders with `MagicNumber` and `counterMagicNumber`.
// - Added logging in `CheckExitConditions` for Main, Counter, and Total Combined P/L.

//--------------------------------------------------------------------
// Version 3.4.0
//--------------------------------------------------------------------
// - Feature: Implemented an Independent Counter Grid.
//   - When any Main Grid order (initial or subsequent grid level) is opened, an opposing
//     Counter Grid order of the same lot size is immediately opened using `counterMagicNumber` (MagicNumber + 1).
//   - The Counter Grid can also add its own grid levels independently if the price moves
//     against its direction by ATR distance, using `LotCoefficient`.
// - New Globals: `counterMagicNumber`, `CTrade counterTrade`, and state variables for the counter grid
//   (`lastCounterBuyPrice`, `lastCounterSellPrice`, `lastCounterBuyLot`, `lastCounterSellLot`,
//   `counterBuyGridCount`, `counterSellGridCount`).
// - `OnInit()`: Initializes `counterMagicNumber` and `counterTrade`.
// - `CheckEntryConditions()`: Opens initial counter trade alongside initial main trade.
// - `ManageGrid()`: Significantly expanded to:
//   - Open counter orders when main grid orders are placed.
//   - Manage independent grid additions for the counter grid.
// - Helper Functions (`PositionsTotalFiltered`, `CalculateFloatingEquity`, `CloseAllPositionsOfType`):
//   Parameterised to accept a `magicNum` argument.
// - `CheckExitConditions()`: Equity Trailing Stop logic continues to operate ONLY on Main Grid P/L.
//   When Main Grid exit is triggered, ALL positions (Main and Counter) are closed.
// - `MaxGridLevelBeforeClose` (Main Grid): If Main Grid hits this level, ALL positions (Main and Counter) are closed.
// - `CheckAndResetGridState()`: Made more complex to handle state resets for Main and Counter grids independently.

//--------------------------------------------------------------------
// Version 3.3.0
//--------------------------------------------------------------------
// - Feature: Replaced EMA Waterfall and previous Equity TP with a new Equity Trailing Stop mechanism.
//   - `EquityTakeProfit` input renamed to `EquityTrailingStep`.
//   - `WaterfallMinProfit` input removed.
// - New Trailing Logic:
//   - Activates when total floating equity (Main Grid only) reaches `EquityTrailingStep * 2.0`.
//   - Initially locks profit at `EquityTrailingStep * 1.0`.
//   - If equity drops to the locked level, all Main Grid positions close.
//   - Stop level steps up by `EquityTrailingStep` for each `EquityTrailingStep` new equity high.
// - New Globals: `isEquityTrailingActive`, `equityTrailStopLevel`, `highestEquitySinceTrailActivated`, `stateNeedsSaving`.
// - New Functions: `SaveTrailingState()`, `LoadTrailingState()` for persistence.
// - `CheckExitConditions()`: Rewritten to implement the new trailing logic.
// - `OnInit()`: Calls `LoadTrailingState()`. Removed waterfall flag resets.
// - `OnDeinit()`: Calls `SaveTrailingState()`.
// - `CheckAndResetGridState()`: Resets new trailing state variables when all Main Grid positions close.
// - Removed EMA Waterfall functions and related global variables.

//--------------------------------------------------------------------
// Version 3.2.4
//--------------------------------------------------------------------
// - Feature: Changed behavior at `MaxGridLevelBeforeHedge`. Instead of hedging,
//   the EA now closes ALL open positions (Main Grid Buys and Sells).
// - `MaxGridLevelBeforeHedge` input renamed to `MaxGridLevelBeforeClose` for clarity.
// - `ManageGrid()`: Modified to call `CloseAllPositionsOfType()` for both BUY and SELL
//   when `buyGridCount` or `sellGridCount` reaches `MaxGridLevelBeforeClose`.
// - Removed Hedging Code:
//   - Deleted global variables `isBuyHedged`, `isSellHedged`.
//   - Deleted `OpenHedgeOrder()` function.
//   - Simplified `CloseAllPositionsOfType()` by removing logic for closing specific hedge orders.
//   - Simplified `CheckAndResetGridState()` by removing hedge flag resets.

//--------------------------------------------------------------------
// Version 3.2.3
//--------------------------------------------------------------------
// - Bug Fix: Corrected EMA Waterfall exit logic for the EMA200 level.
//   - The `WaterfallMinProfit` check is now correctly applied when price reaches
//     EMA200, similar to EMA9/20/50, if the corresponding `waitFor...Ema200` flag is set.
//     Previously, it would close at EMA200 regardless of profit.
// - `CheckExitConditions()`: Modified the EMA200 condition blocks for both Buy and Sell
//   waterfall exits to include the `currentFloatingEquity >= WaterfallMinProfit` check.

//--------------------------------------------------------------------
// Version 3.2.2
//--------------------------------------------------------------------
// - Feature: Prevented further Main Grid orders after its hedge was activated.
// - New Globals: `isBuyHedged`, `isSellHedged` (boolean flags).
// - `OpenHedgeOrder()`: Sets the corresponding `isBuyHedged` or `isSellHedged` to true.
// - `ManageGrid()`: Added checks `!isBuyHedged` and `!isSellHedged` to conditions
//   for adding new Main Grid Buy and Sell orders, respectively.
// - `CheckAndResetGridState()`: Resets `isBuyHedged`/`isSellHedged` to false when
//   the corresponding Main Grid side (Buys or Sells) is fully closed.

//--------------------------------------------------------------------
// Version 3.2.1
//--------------------------------------------------------------------
// - Bug Fix: Corrected `OnInit` syntax errors.
//   - Added missing `InpDeviation` input parameter.
//   - Changed incorrect `iMAOnArray` call for `volumeMaHandle` back to the correct
//     `iMA(_Symbol, _Period, VolumeMAPeriod, 0, MODE_SMA, volumeHandle)`.
//   - Fixed a mismatched brace at the end of `OnInit`.

//--------------------------------------------------------------------
// Version 3.2.0
//--------------------------------------------------------------------
// - Feature: Added Level 5 Hedge for the Main Grid.
//   - When the 5th Main Grid order (Buy or Sell) is opened, an opposing hedge order
//     is placed. Lot size of hedge = total lots of the 5 Main Grid orders.
//   - Input `MaxGridLevelBeforeHedge` controls this level (default 5).
// - New Functions: `CalculateTotalLotsOfType()`, `OpenHedgeOrder()`.
// - `ManageGrid()`: Calls `OpenHedgeOrder()` when `buyGridCount` or `sellGridCount`
//   reaches `MaxGridLevelBeforeHedge`.
// - `CloseAllPositionsOfType()`: Modified to also find and close the corresponding
//   hedge order (identified by comment) when the main grid it was hedging is closed.

//--------------------------------------------------------------------
// Version 3.1.2 (Initial version provided by user for analysis)
//--------------------------------------------------------------------
// - Base EA: Grid trading strategy.
// - Entry: Based on price relation to EMAs (9, 20, 50, 200) and Volume > Volume MA.
// - Grid: Adds levels at ATR-defined distances with `LotCoefficient`.
// - Exits:
//   1. Dynamic Equity Take Profit (per order).
//   2. EMA Waterfall Exit with `WaterfallMinProfit` check.
// - Filters: Spread, Hourly, Friday.
//--------------------------------------------------------------------

