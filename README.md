# NQ Futures ML Risk Audit

[![Reproducibility](https://github.com/braxtonmensah/nq-futures-ml-risk-audit/actions/workflows/ci.yml/badge.svg)](https://github.com/braxtonmensah/nq-futures-ml-risk-audit/actions/workflows/ci.yml)

Independent reconstruction and robustness audit of a machine-learning volatility gate for NQ futures.

An earlier version reported a trade-level t-statistic as a Sharpe-like metric. This audit rebuilds the strategy from raw bars, replaces that number with monthly annualized Sharpe, and tests sensitivity to leakage, slippage, fill ordering, label shuffling, and cross-instrument transfer.

## Key Results

At **1.5 NQ points of slippage per leg**, the reconstructed strategy produced:

| Test | Trades | PnL | Win Rate | Profit Factor | Monthly Ann. Sharpe |
|---|---:|---:|---:|---:|---:|
| Replicated original logic | 2,022 | +$61,301 | 66.6% | 1.38 | 2.85 |
| Clean-eval rerun | 1,925 | +$60,511 | 67.1% | 1.39 | 2.91 |
| Purged clean-eval rerun | 1,923 | +$56,145 | 66.4% | 1.36 | 2.87 |
| Label-shuffle null | 0 | $0 | 0.0% | n/a | n/a |

The clean and purged reruns remained positive, while the label-shuffle control produced no trades. Block-bootstrap samples also remained positive across day, week, and month blocks.

## Stress Tests

The audit also identifies the conditions under which the result breaks:

- Same-bar adverse fill ordering produces **-$175,264**.
- PnL turns negative at **2.5 points of slippage per leg**.
- The same logic does not generalize to ES or MGC.
- The original validation used the test fold for LightGBM early stopping; the clean and purged reruns correct that leakage.

These are material limits. The repository supports an NQ-specific historical result, not a claim of live profitability or broad futures generalization.

## Evidence

![Equity curve at 1.5 pt/leg slippage](overfit_audit/charts/equity_curve_1p5_slippage.png)

![Drawdown at 1.5 pt/leg slippage](overfit_audit/charts/drawdown_1p5_slippage.png)

![Slippage stress breakpoint](overfit_audit/charts/slippage_stress_pnl.png)

## Reproduce

```bash
pip install -r requirements.txt
python src/reproduce_metrics.py
pytest -q
```

The quick reproducer reads committed audit artifacts and does not require vendor data. A full rerun requires one-minute OHLCV files with `timestamp`, `open`, `high`, `low`, `close`, and `volume` columns:

```bash
set NQ_RAW_1M_CSV=C:\path\to\databento_nq_1m.csv
set ES_RAW_1M_CSV=C:\path\to\databento_es_1m.csv
set MGC_RAW_1M_CSV=C:\path\to\databento_mgc_1m.csv
python src\overfit_replication_audit.py
```

## Repository Guide

| Path | Purpose |
|---|---|
| `report.pdf` | Technical report. |
| `overfit_audit/AUDIT_REPORT.pdf` | Independent audit report. |
| `overfit_audit/claim_matrix.csv` | Original claims mapped to audit outcomes. |
| `docs/METHODS.md` | Methodology and validation design. |
| `docs/VALIDATION_CHECKLIST.md` | Reproducibility checklist. |
| `src/reproduce_metrics.py` | Quick artifact reproducer. |
| `src/overfit_replication_audit.py` | Full raw-data audit. |
| `paper_trading_log/` | Locked forward-validation templates. |

## Next Step

The next credible test is a locked paper-trading period with no parameter changes. The templates in `paper_trading_log/` record signals, simulated fills, estimated slippage, and weekly reviews without allowing the backtest to drift after deployment.
