# Minimum-Variance Electricity Price Hedging

## Project Overview

This project explores how electricity price risk in **Western Denmark (DK1)** can be reduced using prices from neighboring European electricity markets.

The objective is to construct a **minimum-variance cross-market proxy hedge** using historical day-ahead electricity prices and portfolio optimization techniques.

The analysis focuses on five electricity bidding zones:

- **DK1:** Western Denmark (target market)
- **DE:** Germany
- **DK2:** Eastern Denmark
- **NO2:** Southern Norway
- **SE4:** Southern Sweden

The project is implemented in Python using historical electricity market data.

## Research Objective

Electricity prices are highly volatile due to changes in demand, renewable energy production, fuel prices and market conditions.

This project investigates whether price movements in neighboring electricity markets can help reduce the exposure to DK1 price fluctuations.

The main research question is:

**Can a minimum-variance combination of neighboring electricity markets significantly reduce DK1 electricity price risk?**

## Dataset

The analysis uses historical hourly day-ahead electricity prices from ENTSO-E exports, covering observations between 2015 and 2025, with the common sample determined by data availability.

The raw data is organized into five market folders:

```text
ElectricityPrices/
    DK1/
        Prices/
    DE/
        Prices/
    DK2/
        Prices/
    NO2/
        Prices/
    SE4/
        Prices/
```

Only the original day-ahead price CSV files are used.

Data preparation includes:

- Reading and combining yearly CSV files.
- Extracting hourly day-ahead electricity prices.
- Converting timestamps to UTC.
- Aligning observations across the five markets.
- Calculating hourly price changes in EUR/MWh.

Hourly absolute price changes are used instead of percentage returns because electricity prices can be negative or close to zero.

## Methodology

### 1. Minimum-Variance Optimization

The objective is to minimize DK1 electricity price risk using four neighboring markets: DE, DK2, NO2, and SE4.

The hedged hourly price change is defined as:

$$
\Delta P_{H,t} = \Delta P_{DK1,t} - \sum_{i=1}^{4} w_i \Delta P_{i,t}
$$

where:
- $\Delta P_{DK1,t}$ represents the hourly price change in DK1.
- $\Delta P_{i,t}$ represents the hourly price change in market $i$.
- $w_i$ represents the hedge weight assigned to market $i$.

The optimization problem is:

$$
\min_{\mathbf{w}} \; \mathrm{Var}\left(\Delta P_{DK1,t} - \sum_{i=1}^{4} w_i \Delta P_{i,t}\right)
$$

Subject to:

$$
0 \leq w_i \leq 1
$$

$$
\sum_{i=1}^{4} w_i \leq 1
$$

The optimization is performed using the **Sequential Least Squares Programming (SLSQP)** algorithm from SciPy.

The resulting weights minimize the variance of DK1's residual hourly price changes, subject to non-negative weights and a maximum total hedge exposure of 100%.

### 2. Train-Test Split

The observations are divided chronologically into:

- **70% training data:** used to estimate the optimal hedge weights.
- **30% testing data:** used to evaluate performance on unseen observations.

### 3. Performance Evaluation

Hedging effectiveness is measured by the percentage reduction in variance:

\[
HE= \left(1-\frac{\operatorname{Var}(\Delta P_H)}
{\operatorname{Var}(\Delta P_{DK1})}\right)\times100
\]

A higher percentage indicates a greater reduction in hourly price-change variance.

## Results

### Optimal Hedge Weights

| Market | Optimal Weight |
|---|---:|
| Germany (DE) | 45.03% |
| Eastern Denmark (DK2) | 40.71% |
| Southern Norway (NO2) | 13.80% |
| Southern Sweden (SE4) | 0.46% |

### Risk Comparison

| Metric | Training | Testing |
|---|---:|---:|
| Observations | 44,284 | 18,980 |
| Unhedged standard deviation | 17.77 | 19.31 |
| Hedged standard deviation | 8.63 | 7.37 |
| Variance reduction | 76.39% | 85.44% |

The results show that the optimized proxy hedge significantly reduces the variance of DK1 hourly price changes.

In the out-of-sample test period, the model achieves a **variance reduction of approximately 85.44%**.

Germany and Eastern Denmark account for most of the optimal hedge exposure.

These results demonstrate strong historical statistical risk reduction, although they do not establish the profitability of an executable trading strategy.

## Python Implementation

The project contains two main scripts: (that i implemented in one ipynb) 

```text
first cell : 01_discover_simple.py
second cell : 02_min_variance_simple.py
```

**01_discover_simple.py**

Explores the original CSV files, identifies their columns and checks the available electricity price data.

**02_min_variance_simple.py**

Loads and prepares the data, performs minimum-variance optimization, evaluates hedging effectiveness and exports the results.

### Requirements

Python 3.10 or later is recommended.

```bash
pip install pandas numpy scipy matplotlib
```

### Running the Project

1. Download or prepare the raw electricity price CSV files.
2. Update the local data directory in the scripts.
3. Run the discovery script.
4. Run the optimization script.

```bash
python 01_discover_simple.py
python 02_min_variance_simple.py
```

The optimization generates a `hedge_results` folder containing the optimal hedge weights, risk comparison, aligned electricity prices and visualization.

## Limitations

This study uses historical spot prices to construct a statistical proxy hedge.

It does not include actual futures or swap prices, transaction costs, contract specifications or realized trading cash flows.

Therefore, the reported variance reductions should be interpreted as **statistical hedging effectiveness**, not as realized financial hedging performance.

Additionally, the current model estimates static hedge weights, which may not remain optimal across different market conditions.

## Future Improvements

Potential extensions include:

- Rolling-window optimization and dynamic hedge weights.
- Conditional Value at Risk (CVaR) optimization.
- Performance analysis during the 2021–2022 European energy crisis.
- Comparison of minimum-variance and tail-risk-based strategies.
- Incorporation of tradable electricity futures data.

## Technologies

Python · Pandas · NumPy · SciPy · Matplotlib

## Author

Independent academic project developed using historical electricity market data collected for a master's thesis.

## Disclaimer

This project is intended for academic research and educational purposes only. The results should not be interpreted as investment or trading recommendations.
