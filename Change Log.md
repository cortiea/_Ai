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
