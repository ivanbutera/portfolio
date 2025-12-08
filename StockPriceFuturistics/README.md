# Stock Price Time Series Forecasting (Adj. Close)

## 1. Project Overview

This project implements and compares three distinct time series models—**ARIMA**, **XGBoost Regressor**, and **Long Short-Term Memory (LSTM) RNN**—to estimate the future price of a stock using its historical data.

### Goal
The primary objective was to find a robust model that can accurately predict the stock's **Adjusted Close Price**, specifically overcoming the tendency of statistical models to revert to a flat-line forecast (known as the "Random Walk" problem in finance).

### Dataset Columns
The model's performance relies solely on the 'Adj\_Close' price. The other columns are:
* **Date**: The trading day.
* **Open, High, Low**: Prices at the market open, the highest point, and the lowest point of the day.
* **Close**: The price at market close (unadjusted).
* **Adj\_Close**: The closing price adjusted for corporate actions (splits, dividends). **This was selected as the target variable for its superior historical accuracy.**
* **Volume**: The total number of shares traded that day.

---

## 2. Methodology & Model Comparison

The time series was split into an **80% Training Set** and a **20% Testing Set** chronologically.

### A. ARIMA (Autoregressive Integrated Moving Average)

| Step | Process | Result | Conclusion |
| :--- | :--- | :--- | :--- |
| **Stationarity ($\mathbf{d}$)** | Augmented Dickey-Fuller (ADF) Test & Differencing. | $\mathbf{d=1}$ (First-order difference required). | Achieved stationarity. |
| **Model Order ($\mathbf{p}, \mathbf{q}$)** | ACF and PACF Plots (or Auto-ARIMA). | $\mathbf{ARIMA(2, 1, 2)}$ was selected. | Model was highly significant (all $p$-values $\le 0.05$). |
| **Diagnostics** | Ljung-Box & Heteroskedasticity Tests. | Ljung-Box passed, but **Heteroskedasticity failed** (P-value $0.00$). | Indicates high, time-varying volatility, suggesting the need for a **GARCH** model. |
| **Prediction** | Out-of-Sample Forecast. | **Flat-line forecast.** | **Failed.** The linear model's memory expired quickly, reverting to the long-term mean due to the data's Random Walk nature. |

### B. XGBoost Regressor

The time series was converted to a supervised learning problem using a **60-day lookback window** as features.

| Metric | Result | Conclusion |
| :--- | :--- | :--- |
| **Initial Tracking** | Excellent short-term fit (R2 initially high). | Model is strong at **interpolation** (predicting within known ranges). |
| **Long-Term Trend** | **Flat-line forecast** after a major price spike. | **Failed.** Tree-based models are poor at **extrapolation** (predicting values outside the training range). The model defaulted to the average target price. |

### C. LSTM (Long Short-Term Memory) RNN

The data was scaled (0 to 1) and sequenced using a **60-day lookback window** as input features.

| Metric | Value | Interpretation |
| :--- | :--- | :--- |
| **R-squared (R2 Score)** | **$\approx 0.9083$** | **Success.** The model explains over $90\%$ of the variance in the stock price during the volatile test period. |
| **Root Mean Squared Error (RMSE)** | $\approx 7.28$ | The average prediction error was $\$7.28$. |
| **Long-Term Trend** | **Successfully tracked the continuation of the aggressive uptrend.** | **Best Performer.** The LSTM's recurrent memory cells successfully captured the long-term non-linear dependencies. |

---

### Future Forecasting Method
The final forecast was generated using a **recurrent (or rolling)** method:
1.  The model predicts the price for Day $T+1$.
2.  The input sequence is shifted: the oldest price (Day $T-59$) is dropped, and the new prediction (Day $T+1$) is appended.
3.  This new 60-day sequence is fed back into the model to predict Day $T+2$.

This ensures the final forecast maintains the current trend captured by the LSTM's successful memory structure.

