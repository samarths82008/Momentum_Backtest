# SPY Momentum Backtest — Built From Scratch

A time-series momentum strategy on SPY, built entirely from scratch in Python with no backtesting libraries. Every line of logic is explicit and explainable.

![Equity Curve](charts/equity_curve.png)

---

## What it does

Tests a simple rule on 11 years of real market data (2015–2026):

> If SPY's trailing 12-month return is positive → stay invested. If negative → go to cash.

Then layers on two enhancements: realistic transaction costs and volatility targeting.

---

## Results

| Strategy | Sharpe | Notes |
|---|---|---|
| Buy & Hold (SPY) | 0.80 | Benchmark |
| Momentum (gross) | 0.67 | Before costs |
| Momentum (net) | 0.64 | After 0.1% per trade |
| Momentum + Vol Target | 0.57 | Risk-scaled position sizing |

**Other metrics:**
- Worst single month: −16.3% (March 2020)
- Strategy correctly went to cash for most of 2022 (rate-hike selloff)
- Rolling Sharpe range: −2.0 (2022–2023) to +3.5 (2017–2018)
- Lookback sensitivity: Sharpe positive across all windows from 21 to 315 days

Buy & Hold won on raw returns. The momentum strategy's value is regime awareness — it stepped aside during the 2022 drawdown and preserved capital when it mattered most.

---

## Charts produced

| Chart | What it shows |
|---|---|
| Equity curve + drawdown + position | Main performance view across all four variants |
| Rolling 252-day Sharpe | How performance changed across market regimes |
| Lookback sensitivity | Sharpe and max drawdown across 7 different signal windows |
| Monthly returns heatmap | Month-by-month breakdown, net of costs |

---

## Key concepts demonstrated

**Lookahead bias prevention**
The single most common backtesting mistake. Using today's signal to make today's trade is illegal information. Fixed with one line:
```python
data['position'] = np.where(data['momentum_signal'].shift(1) > 0, 1, 0)
#                                                          ^^^^^^^^^
#                            Uses YESTERDAY's signal for TODAY's position
```

**Transaction costs**
Assumed 0.1% per trade (realistic for liquid ETFs). Costs are subtracted every time the position changes. Trimmed the Sharpe from 0.67 to 0.64 — small here, but devastating on higher-turnover strategies.

**Volatility targeting**
Position size scales inversely to 20-day realised volatility, targeting 1% daily vol (~16% annualised). Holds less during turbulent markets, more during calm ones. Separates the signal decision (what to trade) from the sizing decision (how much to trade).

```python
data['vol_scale']    = (TARGET_VOL / data['rolling_vol']).clip(upper=1.0)
data['position_vt']  = data['position'] * data['vol_scale'].shift(1)
```

---

## How to run

**Option 1 — Google Colab (recommended)**

1. Download `momentum_backtest.ipynb`
2. Go to [colab.research.google.com](https://colab.research.google.com)
3. File → Upload notebook
4. Run all cells top to bottom

**Option 2 — Local**

```bash
git clone https://github.com/[your-username]/momentum-backtest
cd momentum-backtest
pip install yfinance numpy pandas matplotlib
jupyter notebook momentum_backtest.ipynb
```

---

## Project structure

```
momentum-backtest/
│
├── momentum_backtest.ipynb   # Main notebook — all code and analysis
├── README.md                 # This file
└── charts/                   # Output charts (generated when you run the notebook)
    ├── equity_curve.png
    ├── rolling_sharpe.png
    ├── lookback_sensitivity.png
    └── monthly_heatmap.png
```

---

## Limitations (important)

- Single asset — no cross-sectional diversification across multiple securities
- Long-only — ignores the short leg where additional alpha lives
- 11 years of data — confidence intervals on the Sharpe ratio are wide
- No slippage or market impact modelled beyond flat cost per trade
- Signal degrades in mean-reverting regimes (visible in 2022–2023 rolling Sharpe)
- Volatility targeting capped at 1x — no leverage used

---

## Interview questions this project addresses

1. What is lookahead bias and how do you prevent it?
2. Why does a high gross Sharpe not guarantee a viable strategy?
3. Why choose a 12-month lookback window?
4. How does volatility targeting improve risk-adjusted returns?
5. How would you extend this to a multi-asset portfolio?

Full answers to all five are written into the notebook as markdown cells.

---

## Stack

- Python 3.x
- yfinance — price data
- pandas / numpy — data manipulation and signal logic
- matplotlib — all visualisations

---

## Next steps / extensions

- [ ] Multi-asset version across 10–20 ETFs with cross-sectional ranking
- [ ] Mean reversion strategy for comparison in choppy regimes
- [ ] Risk parity portfolio weighting
- [ ] Factor regression to decompose returns into known risk premia
