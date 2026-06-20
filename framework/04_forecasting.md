# Module 4: AI-Powered Revenue Forecasting

## Overview

Build a marketing-owned revenue forecast that ties channel spend to pipeline and closed revenue — giving you the data to defend budgets, set realistic targets, and optimize mix in real time.

---

## Forecasting Architecture

```
INPUTS                    MODEL                    OUTPUTS
──────                    ─────                    ───────
Channel spend      →                         →    Pipeline forecast
MQL volume         →   Time-series model    →    Revenue forecast
SQL conversion     →    + regression layer   →    ROI by channel
Historical data    →                         →    Scenario plans
Seasonality        →                         →    Budget optimizer
```

---

## Model 1: Pipeline Forecasting with Prophet

```python
from prophet import Prophet
import pandas as pd
import matplotlib.pyplot as plt

def build_pipeline_forecast(pipeline_df, periods=90):
    """
    Forecast marketing pipeline using Facebook Prophet.
    
    pipeline_df: DataFrame with columns:
    - ds: date (daily or weekly)
    - y: pipeline value ($) or MQL count
    - channel_spend: optional regressor
    """
    
    model = Prophet(
        seasonality_mode='multiplicative',
        yearly_seasonality=True,
        weekly_seasonality=True,
        daily_seasonality=False,
        changepoint_prior_scale=0.05,
        interval_width=0.80
    )
    
    if 'channel_spend' in pipeline_df.columns:
        model.add_regressor('channel_spend')
    
    model.fit(pipeline_df)
    future = model.make_future_dataframe(periods=periods, freq='D')
    
    if 'channel_spend' in pipeline_df.columns:
        avg_spend = pipeline_df['channel_spend'].tail(30).mean()
        future['channel_spend'] = avg_spend
    
    forecast = model.predict(future)
    return model, forecast


def scenario_forecast(base_forecast, spend_multipliers):
    scenarios = {}
    for name, multiplier in spend_multipliers.items():
        scenario_fc = base_forecast.copy()
        scenario_fc['yhat'] = base_forecast['yhat'] * (0.5 + 0.5 * multiplier)
        scenarios[name] = scenario_fc
    return scenarios
```

## Model 2: MQL-to-Revenue Bridge

```python
def mqls_to_revenue_forecast(mqls_by_week, conversion_rates, avg_deal_size, sales_cycle_days=45):
    results = []
    for week, mql_count in mqls_by_week.items():
        sql_count = mql_count * conversion_rates['mql_to_sql']
        opp_count = sql_count * conversion_rates['sql_to_close']
        pipeline_value = opp_count * avg_deal_size
        close_week = week + pd.Timedelta(days=sales_cycle_days)
        results.append({
            'mql_week': week, 'mqls': mql_count, 'sqls': sql_count,
            'opportunities': opp_count, 'pipeline': pipeline_value,
            'expected_close_week': close_week, 'expected_revenue': pipeline_value
        })
    return pd.DataFrame(results)
```

---

## Budget Optimization

```python
from scipy.optimize import minimize
import numpy as np

def optimize_channel_budget(total_budget, channel_response_curves, min_spend_per_channel=1000):
    channels = list(channel_response_curves.keys())
    n = len(channels)
    def total_mqls_neg(spends):
        return -sum(channel_response_curves[ch](spends[i]) for i, ch in enumerate(channels))
    constraints = [{'type': 'eq', 'fun': lambda x: sum(x) - total_budget}]
    bounds = [(min_spend_per_channel, total_budget) for _ in channels]
    x0 = [total_budget / n] * n
    result = minimize(total_mqls_neg, x0, method='SLSQP', bounds=bounds, constraints=constraints)
    optimal_allocation = {ch: result.x[i] for i, ch in enumerate(channels)}
    return optimal_allocation, -result.fun

def paid_search_response(spend): return 50 * np.log(1 + spend / 5000)
def content_response(spend): return 0.003 * spend
def paid_social_response(spend): return 40 * (1 - np.exp(-spend / 8000))

channel_curves = {'paid_search': paid_search_response, 'content': content_response, 'paid_social': paid_social_response}
optimal, expected_mqls = optimize_channel_budget(50000, channel_curves)
```

---

## Pilot Structure (30 Days)

**Week 1:** Pull 24 months of MQL + pipeline + close data
**Week 2:** Build Prophet forecast, validate against known actuals
**Week 3:** Layer in MQL-to-revenue bridge, build 90-day forecast
**Week 4:** Present to leadership with 3 scenarios (conservative/base/aggressive)

**Success metrics:**
- Forecast accuracy vs. actuals at 30/60/90 days (target: within ±20%)
- Leadership adoption (does finance use your forecast in planning?)
- Budget decisions influenced by the model
