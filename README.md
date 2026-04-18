# JSE Risk Analyzer — GARCH-Based Volatility & VaR Modeling

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

## Overview

A quantitative risk analysis project applying **GARCH(1,1) volatility modeling** 
to five major Johannesburg Stock Exchange (JSE) stocks. The model is trained on 
historical data (2019–2024) and validated on completely unseen 2025 market data 
using **Value-at-Risk (VaR) backtesting** and the **Kupiec Proportion of Failures test**.

This project demonstrates that standard deviation alone is insufficient for 
modeling African equity market risk — fat-tailed return distributions and 
volatility clustering require more sophisticated approaches.

---

## Stocks Analyzed

| Company | Ticker | Sector |
|---|---|---|
| Naspers | NPN.JO | Technology / Media |
| MTN Group | MTN.JO | Telecommunications |
| Standard Bank | SBK.JO | Financial Services |
| Anglo American | AGL.JO | Mining |
| Shoprite | SHP.JO | Consumer Retail |

---

---

## Methodology

### 1. Why GARCH?
EDA revealed kurtosis values between **4.9 and 9.5** across all stocks 
(normal distribution = 3.0). This confirms fat-tailed return distributions 
where extreme moves occur far more frequently than standard models predict. 
GARCH(1,1) addresses this by modeling **time-varying volatility** — letting 
volatility change each day based on recent shocks.

### 2. GARCH(1,1) Formula
σ²_t = ω + α·ε²_(t-1) + β·σ²_(t-1)
- **ω (omega)** — baseline long-run volatility
- **α (alpha)** — sensitivity to yesterday's return shock
- **β (beta)**  — persistence of yesterday's volatility
- **α + β**     — total persistence (how long shocks last)

### 3. Train / Test Split
| Set | Period | Days |
|---|---|---|
| Training | 2019–2024 | ~1,500 days |
| Test (unseen) | 2025 | 248 days |

Models were trained on 6 years of JSE market data (2019–2024) covering 
multiple market regimes including the COVID crash (2020), post-COVID 
recovery, and the global rate hike cycle (2022–2023). Models were saved 
using `joblib` and loaded fresh for 2025 predictions — no data leakage.

### 4. Value-at-Risk (VaR)
VaR answers: *"What is the maximum loss we expect on X% of trading days?"*

- **95% VaR** — loss threshold exceeded on ~5% of days
- **99% VaR** — loss threshold exceeded on ~1% of days

---

## Key Results

### GARCH(1,1) Parameters

| Stock | ω (omega) | α (alpha) | β (beta) | Persistence (α+β) | Nu (df) |
|---|---|---|---|---|---|
| Naspers | 0.1463 | 0.0576 | 0.9189 | 0.9766 | 5.91 |
| MTN Group | 0.1031 | 0.0435 | 0.9461 | 0.9896 | 3.42 |
| Standard Bank | 0.3563 | 0.0891 | 0.8556 | 0.9447 | 4.68 |
| Anglo American | 0.0309 | 0.0315 | 0.9609 | 0.9924 | 4.88 |
| Shoprite | 0.0033 | 0.0011 | 0.9971 | 0.9982 | 3.94 |

**Key observations:**
- All stocks show persistence > 0.94 confirming that **volatility shocks 
  on the JSE are long-lasting** — a key insight for portfolio risk managers
- **Shoprite** has the highest persistence (0.9982) — once volatility 
  spikes, it takes a very long time to mean-revert
- **MTN Group** has the lowest Nu (3.42) — heaviest tails of all five 
  stocks, meaning extreme daily moves are most frequent
- **Standard Bank** has the highest alpha (0.0891) — most reactive stock, 
  volatility jumps fastest in response to new shocks
- **Anglo American** has the highest beta (0.9609) — once volatile, 
  stays volatile the longest

### Backtest Results (2025 unseen data)
| Stock | Expected 95% Breaches | Actual 95% Breaches | Result |
|---|---|---|---|
| Naspers | 12 | 8 | ✅ Conservative |
| MTN Group | 12 | 5 | ✅ Conservative |
| Standard Bank | 12 | 7 | ✅ Conservative |
| Anglo American | 12 | 7 | ✅ Conservative |
| Shoprite | 12 | 9 | ✅ Conservative |

### Kupiec POF Test
| Stock | VaR 95% | VaR 99% |
|---|---|---|
| Naspers | ✅ PASS (p=0.1715) | ✅ PASS (p=0.7512) |
| MTN Group | ❌ FAIL (p=0.0147) | ✅ PASS (p=0.7480) |
| Standard Bank | ✅ PASS (p=0.0876) | ✅ PASS (p=0.7512) |
| Anglo American | ✅ PASS (p=0.0876) | ✅ PASS (p=0.3730) |
| Shoprite | ✅ PASS (p=0.2986) | ✅ PASS (p=0.3730) |

**9 out of 10 tests passed.** MTN Group at 95% VaR failed because the 
model was overly conservative — only 5 breaches occurred vs 12 expected, 
meaning the model overestimated MTN's risk rather than underestimating it.

### Anglo American Note
Anglo American showed unusually high forecast volatility (~23%) in 2025, 
likely reflecting its ongoing corporate restructuring and partial demerger 
activity during this period. This is a known market event, not a model failure.

---

## How to Reproduce

```bash
# 1. Clone the repo
git clone https://github.com/EWULO/jse-risk-analyzer.git
cd jse-risk-analyzer

# 2. Install dependencies
pip install yfinance pandas numpy matplotlib seaborn arch scipy joblib

# 3. Run notebooks in order
#    01 → 02 → 03 → 04
```

> Note: `.pkl` model files are excluded from git. 
> Run `03_garch_model.ipynb` first to regenerate them.

---

## Key Findings

1. **Fat tails are real** — Kurtosis of 5–9.5 across JSE stocks justifies 
   GARCH over simple standard deviation models
2. **Volatility is persistent** — α+β > 0.94 for all stocks means market 
   shocks take weeks to dissipate on the JSE
3. **GARCH is conservative on JSE stocks** — actual 2025 breaches were 
   consistently below expected, suggesting the model is a reliable 
   and prudent risk management tool
4. **99% VaR is robust** — all 5 stocks passed the Kupiec test at the 
   stricter 99% confidence level
5. **Training on multiple market regimes matters** — 6 years of training 
   data including COVID and rate hike cycles produced well-calibrated 
   out-of-sample forecasts

---

## Tools & Libraries

`Python` `pandas` `numpy` `arch` `scipy` `yfinance` `matplotlib` 
`seaborn` `joblib`

---

## Author
**Oluwarotimi Ewulo**  

