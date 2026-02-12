# Dollar Cost Average (DCA - PAC) + Inflation Adjustment

TradingView indicator written in Pine Script v6 that simulates a **Dollar Cost Averaging (DCA)** plan on any asset available on TradingView, with fixed-amount monthly purchases.

## Features

- Configurable fixed monthly investment amount
- Customizable investment period (start/end date)
- Nominal and percentage P/L calculation
- Annualized return (CAGR)
- Portfolio Max Drawdown
- Capital gains taxation with configurable tax rate
- Inflation adjustment based on real CPI data

## Settings

### DCA Settings

| Parameter | Default | Description |
|-----------|---------|-------------|
| Monthly Amount | 100 | Amount invested each month (minimum 1) |

### Period

| Parameter | Default | Description |
|-----------|---------|-------------|
| Start Date | 2005-12-01 | Start date of the accumulation plan |
| End Date | 2025-12-31 | End date of the accumulation plan |

### Taxation

| Parameter | Default | Description |
|-----------|---------|-------------|
| Capital Gains Tax % | 26.0% | Tax rate on capital gains (default: 26%, Italian rate) |

### Inflation

| Parameter | Default | Description |
|-----------|---------|-------------|
| Enable CPI Adjustment | true | Enable/disable inflation adjustment |
| CPI Country | USA | Reference country for CPI data |

## How to use

1. Add the indicator to any chart on TradingView
2. Configure the monthly amount, period, and tax rate in the settings
3. To see inflation-adjusted values, enable CPI and select the reference country
4. The indicator will display:
   - **Chart**: orange line (nominal invested), green/red line (portfolio value), yellow line (real invested, if CPI is enabled), with a green/red fill area between invested and portfolio value
   - **Summary table** (top right): all summary data with "Nominal" and "Real (CPI)" columns when inflation is enabled

## Summary table

The table displays the following metrics:

| Row | Description |
|-----|-------------|
| Invested | Total invested amount (nominal and real) |
| Portfolio Value | Current portfolio value |
| N. Investments | Number of purchases made |
| Avg Cost | Average cost basis |
| P/L % | Percentage profit/loss |
| P/L | Absolute profit/loss |
| P/L % Ann. | Annualized return (CAGR) |
| Max Drawdown | Maximum peak-to-trough decline (%) |
| Tax Rate | Configured tax rate |
| Tax on P/L | Taxes calculated on profit |
| Net After Taxes | Portfolio value net of taxes |
| CPI | Current CPI index value (if enabled) |

## Inflation adjustment (CPI)

### What is CPI

The CPI (Consumer Price Index) is an index number that measures the average level of consumer prices. It is not a percentage: for example, a CPI of 290 means prices are at 290% relative to the base period (for the USA, the base period is 1982-1984 = 100).

### How the calculation works

The goal is to answer the question: **how much should my portfolio be worth to maintain the same purchasing power as the money I invested?**

The calculation happens in two steps:

**Step 1 — Deflation at purchase time**

At each monthly purchase, the amount is divided by the current CPI:

```
totalInvestedReal += monthlyAmount / CPI_at_purchase
```

**Step 2 — Revaluation at current CPI**

On every bar, the deflated total is multiplied by the current CPI:

```
investedReal = totalInvestedReal × CPI_current
```

### Why it works

For each individual contribution, the formula is equivalent to:

```
real_contribution = monthlyAmount × (CPI_current / CPI_at_purchase)
```

The ratio `CPI_current / CPI_at_purchase` measures exactly the inflation that occurred since the purchase. This means:

- **At the time of purchase**: `CPI_current = CPI_at_purchase`, so the real contribution = nominal amount ($100 is worth $100)
- **After a period of inflation**: if CPI has risen by 10%, the real contribution will be $110 — you need $110 today to buy what $100 could buy at the time of purchase

### Example

| Month | Amount | CPI at purchase | CPI today (320) | Real invested |
|-------|--------|----------------|-----------------|---------------|
| Jan 2020 | $100 | 290 | 320 | 100 × 320/290 = $110.34 |
| Jan 2021 | $100 | 300 | 320 | 100 × 320/300 = $106.67 |
| Jan 2022 | $100 | 310 | 320 | 100 × 320/310 = $103.23 |
| **Total** | **$300** | | | **$320.24** |

The $300 invested nominally is equivalent to $320.24 in today's purchasing power. If the portfolio is worth less than $320.24, the investment has lost real value despite potentially showing a nominal gain.

## CPI indices by country

CPI data is sourced from the Federal Reserve Economic Data (FRED) via TradingView's `request.security()` on a monthly timeframe.

| Country | TradingView Symbol | FRED Series | Description |
|---------|--------------------|-------------|-------------|
| USA | `FRED:CPIAUCSL` | CPIAUCSL | Consumer Price Index for All Urban Consumers: All Items |
| EU (Eurozone) | `FRED:CP0000EZ19M086NEST` | CP0000EZ19M086NEST | Harmonised Index of Consumer Prices: All Items for Euro Area (19 Countries) |
| UK | `FRED:GBRCPIALLMINMEI` | GBRCPIALLMINMEI | Consumer Price Index: All Items for United Kingdom |
| Japan | `FRED:JPNCPIALLMINMEI` | JPNCPIALLMINMEI | Consumer Price Index: All Items for Japan |
| Canada | `FRED:CANCPIALLMINMEI` | CANCPIALLMINMEI | Consumer Price Index: All Items for Canada |
| Australia | `FRED:AUSCPIALLMINMEI` | AUSCPIALLMINMEI | Consumer Price Index: All Items for Australia |

Each index has its own base period and scale, but the adjustment calculation works correctly with any base because it only uses the ratio between CPI values.