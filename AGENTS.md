# SPX Recipe Agent Project

This project builds and optimizes an intraday long/short entry-exit algorithm for S&P 500 futures trading from historical 5-minute OHLCV data.

## What this project does
- Takes historical SPX 5-min bars (`data/SPX_5min.csv`) as input.
- Treats a trading strategy as a RECIPE: ingredients (entry signals, filters, exits) + amounts (parameters).
- Backtests, then adds/subtracts ingredients one at a time until profit factor, win rate, and per-trade gain are maximized — with walk-forward splits and anti-overfitting gates.
- Outputs the frozen recipe, full metrics, and a complete trial log to `output/`.

## How to use
1. Put the full 5-min SPX CSV at `data/SPX_5min.csv` (columns: `time,open,high,low,close,volume`; see `data/SPX_5min_sample.csv` for format).
2. Run opencode in this directory and invoke the agent:
   - `@spx-recipe-chef build the recipe from data/SPX_5min.csv`
   - or select it with Tab and run: `build the recipe from data/SPX_5min.csv`
3. Read `output/report.md` for the verdict (PASS / STRONG PASS / FAIL) and `output/recipe.json` for the final algorithm.

## Agent
The strategy optimizer lives in `.opencode/agents/spx-recipe-chef.md`. Read that file before modifying anything — it is the full executable spec (input schema, workflow, decision rules, quality gates, error handling, ingredient library).

## Configuration
Optional `config.json` overrides defaults (contract specs, costs, split, gates). See the spec in `.opencode/agents/spx-recipe-chef.md` for the full schema.

## Rules for any agent working in this repo
- NEVER tune parameters after seeing out-of-sample results (spec rule R13).
- NEVER shuffle time series; splits are chronological.
- EVERY backtest is logged to `output/trials.csv` — no exceptions.
- Costs (default 0.35 pts/round trip on ES) are applied to every trade, always.
- If you change the data or config, the whole search restarts — never mix results across configurations.

## Context
This project extends the TRACE day-trading research (`day-trading-strategy-TRACE-2026-08-10.md` in the parent workspace): ORB continuation edges, session VWAP, dead-zone vetoes, and the cost budget are baked into the ingredient library. The optimizer's job is to let the data confirm or reject each ingredient on this specific dataset.
