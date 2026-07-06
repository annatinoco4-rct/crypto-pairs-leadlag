# crypto-pairs-leadlag
(in process)

Intra-sector pairs lead-lag statistical arbitrage on crypto perpetual futures.
A methodology-first backtest built to a strict statistical-rigor standard, using public market data only.

> **Status: in development.** This README describes the research design and the rigor standard, which are fixed. Implementation status is tracked explicitly in [Roadmap](#roadmap) — each module is marked *planned* until its code and tests exist. No result is claimed here until it is reported in [Results](#results), net of costs.

---

## Research question

Does a *tradeable* intra-sector lead-lag effect exist between crypto perpetual futures, after costs?

Within a crypto "sector" (e.g. L1s, DeFi), does the short-horizon return of a leading asset predict the move of a co-sector follower? The follower's exposure to the leader is removed via an OLS hedge ratio, so the bet is on the **residual spread**, not on direction. The hypothesis is grounded in gradual information-diffusion arguments (information reaches liquid majors first and propagates to less-liquid co-sector names with a short lag).

This is the structural analog of a classic equities intra-sector pairs lead-lag study, reimplemented from scratch for the 24/7 perpetuals market.

## Rigor standard (non-negotiable)

The verdict is only as credible as the protocol that produced it. Every result in this repo is held to:

- **Deflated Sharpe Ratio (DSR) with multiple-testing correction** over *all* configurations tested. A high raw Sharpe means nothing without it; the parameter grid (thresholds, lag, windows) counts as trials.
- **Walk-forward validation**, not a single in-sample split.
- **Gross/net decomposition.** The verdict is always given *net of costs*.
- **Point-in-time data, no look-ahead.** Every rolling statistic is lagged (`shift(1)`).
- **Dollar-neutral sizing via OLS hedge ratio.** Never equal-weight, which injects directional exposure unless beta = 1.
- **Liquidity gate** at the signal-generation layer.
- **Honest framing.** Proof-of-concept, not a live track record. A statistically significant signal is *not* a tradeable edge. A clean negative result is a valid and reported outcome.

## Crypto-specific pitfalls addressed

- **24/7 market.** No session boundaries or overnight gaps; the definition of a "bar" and a "day" is set explicitly in `Config`.
- **Survivorship bias.** Selecting today's liquid names and pulling their history backward conditions on survival. The current scope uses names liquid across the full window with this limitation stated; a point-in-time universe is noted as future work.
- **Funding rates.** On perpetuals, funding enters net P&L on both legs and is not optional.
- **Fees, slippage, market impact.** Materially higher on alts than on majors; modeled per leg, per side.
- **Wash trading.** Reported volume is treated as an unreliable liquidity proxy.
- **Restated / backfilled exchange data.** A silent source of look-ahead.
- **Liquidity fragmentation across exchanges.** Mitigated by restricting to a single venue in the proof-of-concept.

## Repository structure

```
crypto-pairs-leadlag/
├── README.md
├── .gitignore
├── requirements.txt          # pinned dependencies
├── .python-version           # pyenv pin (3.11.6)
├── notebooks/
│   └── research.ipynb        # development + explanation
├── src/                      # refactored deliverable modules
│   ├── datalayer.py          # CCXT fetch, bar construction, universe
│   ├── pairselector.py       # sectors, candidate pairs, liquidity gate
│   ├── signalengine.py       # rolling OLS hedge ratio, spread, lead-lag signal
│   ├── backtester.py         # dollar-neutral sizing, costs + funding, walk-forward
│   └── analytics.py          # equity, gross/net, Sharpe, DSR, plots
├── run_backtest.py           # orchestrator (run from project root)
├── results/                  # curated final artifacts (committed): report + key figures
├── data/                     # local cache of fetched public data (NOT committed)
└── output/                   # scratch run artifacts (NOT committed)
```

The single source of truth for all parameters (universe, bar size, period, cost model, verdict thresholds) is the `Config` object in the notebook / `src`. This README intentionally does not duplicate specific values.

## Data

No market data is committed. All inputs are public OHLCV data fetched locally via CCXT from a single exchange. `data/` and `output/` are git-ignored. To reproduce, run the data layer (see `notebooks/research.ipynb`), which populates `data/` from the exchange.

## Environment

- macOS, Python 3.11.6 (pyenv; pinned in `.python-version`).
- Core stack: `pandas`, `numpy`, `scipy`. `ccxt` for data. DuckDB/SQL optional for the data layer.

```bash
pyenv local 3.11.6
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

## Roadmap

| Component        | Status      | Notes |
|------------------|-------------|-------|
| Config / specs   | Done        | Universe, bar, period, costs, verdict thresholds defined |
| `datalayer`      | Planned     | CCXT fetch, 5m bar construction, point-in-time handling |
| `pairselector`   | Planned     | Sectors, candidate pairs, liquidity gate |
| `signalengine`   | Planned     | Rolling OLS hedge ratio (lagged), spread, lead-lag signal |
| `backtester`     | Planned     | Dollar-neutral sizing, fees + slippage + funding, walk-forward |
| `analytics`      | Planned     | Gross/net, Sharpe, DSR with `n_trials`, plots |
| Final report     | Planned     | Verdict net of costs, DSR over all trials |
| Point-in-time universe (survivorship fix) | Future work | Reconstructed historical universe |
| Multi-exchange   | Future work | Out of scope for proof-of-concept |

## Results

To be reported here once the pipeline is complete: the verdict (net of costs), the DSR over all configurations tested, walk-forward equity, and gross-vs-net decomposition. Per the rigor standard, a negative net result is an expected and acceptable outcome and will be reported as such.

## Scope and disclaimer

Independent research on public data. Proof-of-concept, not investment advice and not a live track record. A significant in-sample signal does not imply a deployable edge.
