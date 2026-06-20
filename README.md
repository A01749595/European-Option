# European Option Pricing 

Valuation of European call and put options on **Mercedes-Benz Group (MBG.DE)** using three modeling approaches, with a focus on how different assumptions about price dynamics and volatility affect the estimated option value.

---

## Overview

This project estimates the fair value of a European option written on a real underlying asset (Mercedes-Benz stock) and compares three valuation methods side by side under a **single, shared set of parameters** so the comparison is consistent.

- **Black-Scholes** — closed-form pricing under constant volatility and geometric Brownian motion.
- **Monte Carlo + ARCH** — path simulation driven by an ARCH(1) model of *conditional* (time-varying) volatility.
- **Markov chains** — simulation built from an empirical transition matrix estimated directly from historical returns.

The underlying data is daily price history for MBG.DE pulled from Yahoo! Finance, covering **May 2023 to October 2024**. The notebook walks from raw data download and exploratory analysis through statistical validation, model fitting, simulation, and a final cross-method comparison.

The comparison is deliberately **illustrative and methodological**: estimates are *not* validated against observed market option prices and are *not* computed under a strict risk-neutral measure. The goal is to show how the treatment of volatility and price dynamics changes the resulting valuation, not to declare one method definitively "more accurate."

---

## Problem

Although the Black-Scholes model is the conventional approach for valuing European options, it depends on a significant assumption: that volatility stays fixed throughout the option's life. In practice, financial assets tend to show volatility clustering, a pattern that a constant-volatility model simply cannot reproduce.

---

## Objectives

1. Implement the Black-Scholes formula for European call and put options and compute the associated option Greeks.
2. Model conditional volatility with an **ARCH** process and feed it into a **Monte Carlo** simulation of future prices.
3. Build a **Markov-chain** model of returns and price the option from simulated terminal prices.
4. Run all three methods under one shared parameter set and **compare** the resulting call/put values.

---

## Methods / Models

### 1. Black-Scholes
Closed-form valuation of European call and put options using the five classic inputs (spot price `S`, strike `K`, time to maturity `T`, risk-free rate `r`, volatility `σ`). Volatility is annualized from historical log returns. The full set of **Greeks** — Delta, Gamma, Vega, Theta, and Rho — is implemented and their sensitivity to volatility is visualized.

### 2. Monte Carlo with ARCH volatility
An **ARCH(1)** model is fitted to percentage returns to estimate conditional mean and variance over the forecast horizon. Those forecasts seed a Monte Carlo engine that generates thousands of price trajectories; the option is then valued as the discounted average payoff at maturity. The ARCH fit is validated with residual diagnostics before use.

### 3. Markov chains
Historical log returns are discretized into **five states via quantiles** (each state holding a similar share of observations). An **empirical transition matrix** is estimated by counting day-to-day state transitions and normalizing by row. Starting from the most recent observed state, the model simulates many price paths — each step samples the next state, maps it to a representative return, and updates the price — and prices the option from the resulting terminal-price distribution. Unlike Black-Scholes, this approach assumes neither constant volatility nor normally distributed returns.

---

## Key Technical Features

- **Exploratory analysis**: OHLC chart, log-return time series, return and price histograms, box plots, and normal-fit overlays (interactive Plotly figures).
- **Outlier handling** with both IQR and mean ± 2σ rules, plus before/after distribution comparisons.
- **Statistical validation**: Shapiro-Wilk and Jarque-Bera normality tests, Augmented Dickey-Fuller stationarity test, and an Engle ARCH-effects test (`het_arch`) on model residuals.
- **Geometric Brownian Motion** simulation with correct time-step scaling (`dt = T/nsteps`, diffusion scaled by `√dt`), and best-fit trajectory selection by MSE / MAPE / MAE.
- **Full Greeks suite** (Delta, Gamma, Vega, Theta, Rho) with volatility-sensitivity subplots.
- **Train/test split** for ARCH fitting and forecasting.
- **Reproducible Monte Carlo and Markov simulations** (fixed seed) with payoff calculation and present-value discounting.
- **Side-by-side comparison** delivered as both a formatted text table and a grouped bar chart across the three methods.


---

## Results / Key Findings

- All three methods successfully produce call and put valuations under the shared parameter set; the notebook reports them in a unified comparison table and bar chart.
- **Black-Scholes** prices from a single constant annualized volatility, while **Monte Carlo (ARCH)** and **Markov chains** build their prices from simulated trajectories whose dynamics come, respectively, from estimated conditional volatility and from the empirical transition matrix.
- The gaps between the three prices therefore isolate **the effect of modeling price dynamics versus assuming constant volatility**.
- The simulation-based methods add value chiefly by producing a **full distribution of possible future prices** rather than a single scenario, which helps frame the uncertainty around the option's value — at the cost of more computation and some run-to-run variability.
- Because no risk-neutral measure is imposed and no comparison is made to traded option prices, the study draws a **methodological** rather than a definitive conclusion: it demonstrates *how* assumptions move the valuation, not which method is empirically "correct."

---

## Future Work

- **Risk-neutral simulation** — simulate under a risk-neutral measure to obtain a genuinely arbitrage-free price.
- **Market validation** — benchmark the estimates against real, observed market prices for the options.
- **Richer Markov model** — increase the number of states in the Markov chain for finer return resolution.
- **GARCH extension** — compare the ARCH model against GARCH and related extensions to assess improvements in volatility fit.
