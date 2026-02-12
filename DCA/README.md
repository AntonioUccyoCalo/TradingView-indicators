# Dollar Cost Average (DCA - PAC) + Inflation & FX Adjustment

TradingView indicator written in Pine Script v6 that simulates a **Dollar Cost Averaging (DCA)** plan on any asset available on TradingView, with fixed-amount monthly purchases.

## Features

- Configurable fixed monthly investment amount
- Customizable investment period (start/end date)
- Nominal and percentage P/L calculation
- Annualized return (CAGR)
- Portfolio Max Drawdown
- Capital gains taxation with configurable tax rate
- Inflation adjustment based on real CPI data (multi-country)
- Currency / FX Risk adjustment with automatic conversion to home currency
- Combined FX + CPI analysis for full real-return assessment

## Settings

### DCA Settings

| Parameter      | Default | Description                            |
| -------------- | ------- | -------------------------------------- |
| Monthly Amount | 100     | Amount invested each month (minimum 1) |

### Period

| Parameter  | Default    | Description                         |
| ---------- | ---------- | ----------------------------------- |
| Start Date | 2005-12-01 | Start date of the accumulation plan |
| End Date   | 2025-12-31 | End date of the accumulation plan   |

### Taxation

| Parameter           | Default | Description                                            |
| ------------------- | ------- | ------------------------------------------------------ |
| Capital Gains Tax % | 26.0%   | Tax rate on capital gains (default: 26%, Italian rate) |

### Inflation

| Parameter             | Default | Description                                                            |
| --------------------- | ------- | ---------------------------------------------------------------------- |
| Enable CPI Adjustment | false   | Enable/disable inflation adjustment                                    |
| CPI Country           | USA     | Reference country for CPI data (USA, EU, UK, Japan, Canada, Australia) |

### Currency / FX Risk

| Parameter                       | Default | Description                                                                                                                                             |
| ------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Enable FX Adjustment            | false   | Enable/disable currency risk adjustment                                                                                                                 |
| Asset Currency                  | USD     | Currency in which the asset is denominated                                                                                                              |
| Home Currency                   | EUR     | Your local currency                                                                                                                                     |
| Monthly Amount in Home Currency | false   | If enabled, the Monthly Amount is expressed in your home currency and gets converted to asset currency at each purchase using the current exchange rate |

Supported currencies: USD, EUR, GBP, JPY, CAD, AUD, CHF.

## How to use

1. Add the indicator to any chart on TradingView
2. Configure the monthly amount, period, and tax rate in the settings
3. Optionally enable **CPI** to see inflation-adjusted values and select the reference country
4. Optionally enable **FX Adjustment** to convert results into your home currency
5. The indicator will display:
   - **Chart**: orange line (nominal invested), green/red line (portfolio value), yellow line (real invested via CPI), purple line (invested in home currency via FX), with a green/red fill area between invested and portfolio value
   - **Summary table** (top right): all summary data with dynamic columns depending on which adjustments are enabled

## Summary table

The table displays the following metrics:

| Row             | Description                                               |
| --------------- | --------------------------------------------------------- |
| Invested        | Total invested amount (nominal, FX-adjusted, and/or real) |
| Portfolio Value | Current portfolio value (in asset and/or home currency)   |
| N. Investments  | Number of purchases made                                  |
| Avg Cost        | Average cost basis per share                              |
| P/L %           | Percentage profit/loss                                    |
| P/L             | Absolute profit/loss                                      |
| P/L % Ann.      | Annualized return (CAGR)                                  |
| Max Drawdown    | Maximum peak-to-trough decline (%)                        |
| Tax Rate        | Configured tax rate                                       |
| Tax on P/L      | Taxes calculated on profit                                |
| Net After Taxes | Portfolio value net of taxes                              |
| FX (Asset/Home) | Exchange rate at start and current (if FX enabled)        |
| CPI             | CPI index value at start and current (if CPI enabled)     |

## Currency / FX Risk adjustment

### How it works

The FX adjustment converts portfolio results from the asset's currency to the investor's home currency, showing the real return including exchange rate fluctuations.

### Monthly Amount in Home Currency

When this option is enabled, the monthly investment amount is treated as being in the home currency. At each purchase, it is converted to the asset currency using the current exchange rate:

```
investAmountAssetCcy = monthlyAmount / homePerPortfolio
```

This simulates the common real-world scenario where an investor invests a fixed amount in their local currency each month.

### FX data sources

| Pair type                 | Symbols                                           | Conversion                |
| ------------------------- | ------------------------------------------------- | ------------------------- |
| Direct (USD per 1 unit)   | `FX_IDC:EURUSD`, `FX_IDC:GBPUSD`, `FX_IDC:AUDUSD` | Used as-is                |
| Inverse (units per 1 USD) | `FX_IDC:USDJPY`, `FX_IDC:USDCAD`, `FX_IDC:USDCHF` | Inverted in code (1/rate) |

### FX-specific metrics

When FX is enabled, the table shows an additional column "FX Adj." with:

- Invested amount converted to home currency
- Portfolio value in home currency
- P/L, P/L%, CAGR, Max Drawdown — all in home currency terms
- Taxes and net value in home currency

### Combined FX + CPI

When both FX and CPI are enabled, the last column "FX+CPI" shows the combined effect of currency fluctuations and inflation. This answers: **what is my real return in my home currency, adjusted for local purchasing power?**

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

| Month     | Amount   | CPI at purchase | CPI today (320) | Real invested           |
| --------- | -------- | --------------- | --------------- | ----------------------- |
| Jan 2020  | $100     | 290             | 320             | 100 × 320/290 = $110.34 |
| Jan 2021  | $100     | 300             | 320             | 100 × 320/300 = $106.67 |
| Jan 2022  | $100     | 310             | 320             | 100 × 320/310 = $103.23 |
| **Total** | **$300** |                 |                 | **$320.24**             |

The $300 invested nominally is equivalent to $320.24 in today's purchasing power. If the portfolio is worth less than $320.24, the investment has lost real value despite potentially showing a nominal gain.

## CPI indices by country

CPI data is sourced from the Federal Reserve Economic Data (FRED) via TradingView's `request.security()` on a monthly timeframe.

| Country       | TradingView Symbol        | FRED Series        | Description                                                                 |
| ------------- | ------------------------- | ------------------ | --------------------------------------------------------------------------- |
| USA           | `FRED:CPIAUCSL`           | CPIAUCSL           | Consumer Price Index for All Urban Consumers: All Items                     |
| EU (Eurozone) | `FRED:CP0000EZ19M086NEST` | CP0000EZ19M086NEST | Harmonised Index of Consumer Prices: All Items for Euro Area (19 Countries) |
| UK            | `FRED:GBRCPIALLMINMEI`    | GBRCPIALLMINMEI    | Consumer Price Index: All Items for United Kingdom                          |
| Japan         | `FRED:JPNCPIALLMINMEI`    | JPNCPIALLMINMEI    | Consumer Price Index: All Items for Japan                                   |
| Canada        | `FRED:CANCPIALLMINMEI`    | CANCPIALLMINMEI    | Consumer Price Index: All Items for Canada                                  |
| Australia     | `FRED:AUSCPIALLMINMEI`    | AUSCPIALLMINMEI    | Consumer Price Index: All Items for Australia                               |

Each index has its own base period and scale, but the adjustment calculation works correctly with any base because it only uses the ratio between CPI values.
