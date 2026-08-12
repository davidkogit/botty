# SPX Recipe Chef — Intraday Strategy Recipe Optimizer (opencode agent)

An [opencode](https://opencode.ai) agent that takes historical 5-minute SPX OHLCV data and **builds an intraday long/short entry-exit algorithm for futures trading by treating the strategy as a recipe** — adding and subtracting ingredients (entry signals, filters, exits) until profit factor, win rate, and per-trade gain are maximized, with walk-forward anti-overfitting gates.

## Files

| File | Purpose |
|---|---|
| `.opencode/agents/spx-recipe-chef.md` | The agent definition (full executable spec: identity, input/output schema, 8-step workflow, decision rules, quality gates, error handling, ingredient library) |
| `AGENTS.md` | Project instructions loaded in every opencode session |
| `config.json` | Default configuration (contract specs, costs, split, gates) |
| `data/SPX_5min_sample.csv` | FORMAT SAMPLE ONLY (~60 bars). Replace with the full dataset (≈26k bars, Oct 2 – Dec 31 2025) from the chat attachment |
| `output/` | Created on first run: recipe.json, report.md, trials.csv, trades.csv, equity.csv |

## Setup

1. `cd spx-recipe-agent`
2. Save the full attached dataset as `data/SPX_5min.csv` (columns: `time,open,high,low,close,volume`).
3. Run: `opencode` (or `opencode run`), then invoke the agent:
   - `@spx-recipe-chef build the recipe from data/SPX_5min.csv`
   - or press Tab to select the agent, then give the same instruction.

The agent will write `backtest.py`, `ingredients.py`, `optimize.py`, `run.py`, run the search, and deliver `output/recipe.json` + `output/report.md`.

## What the agent does (in one paragraph)

Splits the data chronologically 70/15/15 (search / validation / out-of-sample). Starts from a base recipe, then in each cycle tries ADDING every unused ingredient, REFINING parameters, and REMOVING dead weight — adopting a change only if profit factor, win rate, and per-trade gain (composite score, weighted 50/25/25) improve ≥ 3% **and** hard gates pass (PF ≥ 1.05, ≥ 100 trades, drawdown ≤ 25% of gross profit). After every cycle the recipe is checked on the validation slice, and if validation degrades > 5% the cycle is reverted and the search stops. The final frozen recipe is run once on the untouched out-of-sample slice and given a PASS / STRONG PASS / FAIL verdict. Every backtest is logged to `output/trials.csv`.

## Key design decisions

- **No lookahead**: signals at bar close → entries at next bar open; stops/targets filled intra-bar via high/low (stop assumed first if both hit).
- **Costs always applied**: default 0.35 points per round trip (≈ $4.50/RT + 1 tick on ES, per your TRACE cost budget). A recipe must beat costs — win rate alone is never enough.
- **Research-informed ingredient library**: ORB with midpoint-direction filter, double-break join, session VWAP, dead-zone veto (11:30–14:00), power-hour continuation, gap filter, ADX as veto only — the TRACE evidence base — alongside classic ingredients (MA/RSI/MACD/Bollinger/Donchian) that the search is free to test and reject.
- **Anti-overfitting**: chronological splits, validation guard, minimum trade counts, parameter grids instead of free tuning, conservative same-bar stop assumption, and a mandatory creative-escape step that can invent new OHLCV-only ingredients if the search stalls.
- **Honest OOS**: the last 15% of data is never touched during search; if OOS fails, the verdict is FAIL and the agent says so.

## Configuration (config.json overrides)

Key options: `point_value` (50 = ES, 5 = MES), `round_trip_cost_points` (0.35 default, 0.75 conservative), `session_end` (default 15:55 ET, no overnight holds), `split`, `max_cycles`, gate thresholds (`min_trades_search`, `adopt_improvement_pct`, `validation_guard_pct`, `max_dd_ratio`), and composite `weights`/`caps`. Full schema in the agent spec.

## Notes & caveats

- **Version note**: this uses the opencode V2 location `.opencode/agents/`. If you're on an older opencode release, move the file to `.opencode/agent/spx-recipe-chef.md` (singular).
- The attached data is ~3 months; the OOS slice is ~2 weeks. Treat any PASS as a research result — extend history to 6–12 months and paper trade before live capital.
- OHLCV only: no tick/order-flow data (CVD-style ingredients are intentionally excluded; CLV proxies are provided instead).
- If a long run hits opencode's step cap, send `continue` to resume the loop.
- This is research software, not financial advice. Day trading carries a high risk of loss.
