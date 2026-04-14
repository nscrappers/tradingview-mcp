# Changelog

All notable changes to this project will be documented in this file.

## [0.7.0] - 2026-03-29

### Added
- **Walk-Forward Backtesting** (`walk_forward_backtest_strategy`):
  - Splits data into N folds (train/test) to validate strategy on unseen forward data
  - Per-fold in-sample vs out-of-sample return comparison
  - **Robustness score** (test/train ratio): ROBUST ≥ 0.8 | MODERATE ≥ 0.5 | WEAK ≥ 0.2 | OVERFITTED < 0.2
  - Aggregate out-of-sample metrics: Sharpe, win rate, max drawdown, total return
  - Supports 2–10 splits, configurable train ratio, both 1d and 1h intervals
- **Full Trade Log** (`include_trade_log=True`):
  - Per-trade breakdown: entry/exit date & price, holding days, gross/net return %, cost %
  - Running capital and cumulative return at each trade
- **Equity Curve** (`include_equity_curve=True`):
  - Capital value + drawdown % at each trade exit — ready for charting
- **1h (Hourly) Timeframe** (`interval="1h"`):
  - All strategies and compare now support intraday hourly data
  - Sharpe ratio annualization corrected for 1h bars (252 × 6 trading hours)
  - Works on `backtest_strategy`, `compare_strategies`, and `walk_forward_backtest_strategy`

### Changed
- `backtest_strategy` tool: added `interval`, `include_trade_log`, `include_equity_curve` params
- `compare_strategies` tool: added `interval` param; now documents all 6 strategies (was 4)
- `run_backtest()` now returns last 5 trades always (`recent_trades`) for quick inspection
- Sharpe ratio calculation now uses interval-aware annualization factor

### Notes (personal)
- I bumped the default `n_splits` for walk-forward from 5 to 4 in my local config — 5 folds felt too granular on shorter date ranges and produced noisy robustness scores.
- Changed default `train_ratio` from 0.7 to 0.75 — gives the in-sample window a bit more data on the 1–2 year ranges I typically test, which stabilizes the Sharpe estimates noticeably.
- Changed default `initial_capital` from 10000 to 100000 in my local setup — easier to reason about position sizes and commission impact at a more realistic account size.
- Bumped default `commission` from 0.001 (0.1%) to 0.0015 (0.15%) to better reflect the actual fees I pay on my broker; was underestimating costs on high-frequency signals.
- Changed default `slippage` from 0.001 to 0.0005 — my limit orders typically fill closer to the signal price, so the original default was too conservative for my workflow.
- Note to self: the robustness score thresholds (0.8/0.5/0.2) feel a bit generous for mean-reversion strategies — considering tightening ROBUST to ≥ 0.9 for those specifically.
- Reconsidering the 252 × 6 annualization factor for 1h bars — in practice I see closer to 6.5 trading hours on US markets, so 252 × 6.5 = 1638 might be more accurate. Will test and update if Sharpe values look off.

---

## [0.6.0] - 2026-03-29

### Added
- **Backtesting Engine v2** (`backtest_strategy`, `compare_strategies`):
  - 6 trading strategies: RSI, Bollinger Band, MACD, EMA Cross, **Supertrend** (🔥 trending 2025), **Donchian Channel** (Turtle Trader classic)
  - Institutional-grade metrics: Sharpe Ratio, Calmar Ratio, Expectancy, Profit Factor, Max Drawdown
  - Transact
