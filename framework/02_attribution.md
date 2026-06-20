# Module 2: AI-Powered Attribution

## Overview

Replace last-touch guesswork with a data-driven attribution model that correctly credits every touchpoint in the buyer journey — and makes budget decisions obvious.

---

## The Problem With Last-Touch

Last-touch attribution gives 100% credit to the final touchpoint before conversion. This systematically over-credits bottom-funnel channels (branded search, retargeting) and under-credits awareness channels (organic content, social, podcast).

The result: brands cut the channels that actually drive demand and over-invest in channels that just capture it.

---

## Attribution Model Comparison

| Model | How it works | Best for | Weakness |
|-------|-------------|----------|----------|
| Last touch | 100% to final touch | Simple reporting | Ignores the journey |
| First touch | 100% to first touch | Awareness measurement | Ignores nurture |
| Linear | Equal credit to all | Balanced baseline | Not accurate |
| Time decay | More credit to recent | Short sales cycles | Undervalues awareness |
| **Data-driven** | **ML-weighted by actual impact** | **Accurate ROI** | **Needs data volume** |
| **Shapley value** | **Game-theory optimal allocation** | **Complex journeys** | **Computationally intensive** |

---

## Building a Data-Driven Attribution Model

### Data Requirements
- Minimum 1,000 conversions per 90-day window
- Complete touchpoint data (UTMs, session IDs, user IDs)
- Clean channel taxonomy (paid search, organic, email, etc.)

### Python Implementation: Shapley Value Attribution

```python
import pandas as pd
import numpy as np
from itertools import combinations

def shapley_attribution(journeys_df):
    """
    Calculate Shapley value attribution for marketing channels.
    
    Parameters:
    journeys_df: DataFrame with columns ['journey_id', 'channels', 'converted']
                 where 'channels' is a list of touchpoints in order
    
    Returns:
    DataFrame with Shapley values per channel
    """
    
    # Get all unique channels
    all_channels = set()
    for channels in journeys_df['channels']:
        all_channels.update(channels)
    all_channels = list(all_channels)
    
    # Calculate conversion rate for each coalition of channels
    def coalition_value(coalition):
        coalition_set = set(coalition)
        mask = journeys_df['channels'].apply(
            lambda x: bool(set(x) & coalition_set)
        )
        subset = journeys_df[mask]
        if len(subset) == 0:
            return 0.0
        return subset['converted'].mean()
    
    # Compute Shapley values
    shapley_values = {}
    n = len(all_channels)
    
    for channel in all_channels:
        other_channels = [c for c in all_channels if c != channel]
        shapley = 0.0
        
        for size in range(len(other_channels) + 1):
            for coalition in combinations(other_channels, size):
                coalition = list(coalition)
                weight = (np.math.factorial(size) * np.math.factorial(n - size - 1)) / np.math.factorial(n)
                
                val_with = coalition_value(coalition + [channel])
                val_without = coalition_value(coalition)
                
                shapley += weight * (val_with - val_without)
        
        shapley_values[channel] = shapley
    
    # Normalize to percentages
    total = sum(shapley_values.values())
    result = pd.DataFrame([
        {'channel': ch, 'shapley_value': v, 'attribution_pct': v/total*100}
        for ch, v in shapley_values.items()
    ]).sort_values('attribution_pct', ascending=False)
    
    return result


# Example usage
journeys = pd.DataFrame({
    'journey_id': [1, 2, 3, 4, 5],
    'channels': [
        ['organic_search', 'email', 'paid_search'],
        ['social', 'organic_search', 'direct'],
        ['paid_search', 'email'],
        ['organic_search', 'paid_search'],
        ['social', 'email', 'paid_search']
    ],
    'converted': [1, 1, 0, 1, 1]
})

attribution = shapley_attribution(journeys)
print(attribution)
```

### Incrementality Testing

Attribution models tell you correlation. Incrementality tells you causation.

```python
# Simple holdout test design
def design_incrementality_test(channel, budget, test_duration_days=14):
    """
    Design a geo-based or user-based holdout test for a marketing channel.
    """
    test_design = {
        'channel': channel,
        'test_group': '70% of audience/geos — see the channel as normal',
        'holdout_group': '30% of audience/geos — channel suppressed',
        'duration': f'{test_duration_days} days',
        'minimum_conversions_needed': 200,
        'primary_metric': 'conversion rate lift',
        'secondary_metrics': ['revenue per user', 'LTV proxy'],
        'significance_threshold': 0.95,
        'budget_impact': f'~{budget * 0.3:.0f} budget held back during test'
    }
    return test_design
```

---

## Attribution Audit Checklist

Before building a new model, audit your current state:

- [ ] UTM parameters applied consistently to all paid channels
- [ ] Cross-device tracking in place (or explicitly scoped out)
- [ ] Offline conversion import configured (for B2B: CRM sync)
- [ ] View-through windows defined and documented
- [ ] Channel taxonomy standardized across all tools
- [ ] Attribution window defined (7-day, 30-day, 90-day)

---

## Pilot Structure (30 Days)

**Week 1:** UTM audit and taxonomy cleanup  
**Week 2:** Pull 90 days of touchpoint data, build baseline model  
**Week 3:** Run Shapley model, compare vs. current attribution  
**Week 4:** Present findings to leadership, propose budget reallocation

**Success metrics:**
- Model coverage (% of conversions with complete journey data, target: 80%+)
- Attribution delta (how different is Shapley vs. last-touch)
- Budget reallocation hypothesis (specific channels to shift spend)
- 90-day revenue impact of reallocation (measured post-test)
