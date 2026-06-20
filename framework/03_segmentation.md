# Module 3: ML-Driven Audience Segmentation

## Overview

Stop marketing to "everyone." ML-driven segmentation identifies the behavioral patterns that predict conversion, retention, and LTV — so you can target the right message to the right person at the right moment.

---

## Segmentation Models by Use Case

| Use Case | Model | What it predicts |
|----------|-------|-----------------|
| Email targeting | RFM scoring | Who to contact and when |
| Ad audiences | Lookalike propensity | Who looks like your best customers |
| Churn prevention | Churn propensity | Who is about to leave |
| Upsell campaigns | LTV tier | Who is worth investing in |
| Content personalization | Behavioral clustering | What content resonates |

---

## Model 1: RFM Segmentation

RFM (Recency, Frequency, Monetary) is the fastest-to-implement model with immediate business value.

```python
import pandas as pd
import numpy as np
from datetime import datetime

def rfm_segmentation(transactions_df, analysis_date=None):
    """
    Calculate RFM scores and segments from transaction data.
    
    transactions_df: DataFrame with columns ['customer_id', 'transaction_date', 'revenue']
    """
    if analysis_date is None:
        analysis_date = datetime.now()
    
    # Calculate RFM metrics
    rfm = transactions_df.groupby('customer_id').agg({
        'transaction_date': lambda x: (analysis_date - x.max()).days,  # Recency
        'customer_id': 'count',  # Frequency
        'revenue': 'sum'  # Monetary
    }).rename(columns={
        'transaction_date': 'recency',
        'customer_id': 'frequency',
        'revenue': 'monetary'
    })
    
    # Score 1-5 (5 = best)
    rfm['r_score'] = pd.qcut(rfm['recency'], 5, labels=[5,4,3,2,1])
    rfm['f_score'] = pd.qcut(rfm['frequency'].rank(method='first'), 5, labels=[1,2,3,4,5])
    rfm['m_score'] = pd.qcut(rfm['monetary'], 5, labels=[1,2,3,4,5])
    
    rfm['rfm_score'] = rfm['r_score'].astype(str) + rfm['f_score'].astype(str) + rfm['m_score'].astype(str)
    
    # Segment mapping
    def segment(row):
        r, f, m = int(row['r_score']), int(row['f_score']), int(row['m_score'])
        if r >= 4 and f >= 4 and m >= 4:
            return 'Champions'
        elif r >= 3 and f >= 3:
            return 'Loyal Customers'
        elif r >= 4 and f <= 2:
            return 'Recent Customers'
        elif r <= 2 and f >= 3 and m >= 3:
            return 'At Risk'
        elif r == 1 and f == 1:
            return 'Lost'
        else:
            return 'Potential Loyalists'
    
    rfm['segment'] = rfm.apply(segment, axis=1)
    
    return rfm


# Example activation strategy per segment
SEGMENT_STRATEGY = {
    'Champions': {
        'action': 'Referral program, VIP access, early product releases',
        'email_frequency': 'Weekly',
        'ad_spend': 'Suppression (already loyal, save budget)'
    },
    'Loyal Customers': {
        'action': 'Upsell to higher tier, loyalty rewards',
        'email_frequency': 'Bi-weekly',
        'ad_spend': 'Lookalike seed audience'
    },
    'At Risk': {
        'action': 'Win-back campaign, personal outreach, special offer',
        'email_frequency': 'Immediate win-back sequence',
        'ad_spend': 'Retargeting with strong offer'
    },
    'Lost': {
        'action': 'Survey to understand why, sunset or HAIL MARY offer',
        'email_frequency': 'One final campaign then suppress',
        'ad_spend': 'Exclude from targeting'
    }
}
```

## Model 2: Behavioral Clustering (K-Means)

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt

def behavioral_clustering(features_df, n_clusters=5):
    """
    Cluster customers by behavioral features.
    
    features_df: DataFrame with behavioral features like:
    - pages_per_session, avg_session_duration, content_category_preference,
      email_open_rate, feature_usage_count, etc.
    """
    scaler = StandardScaler()
    X_scaled = scaler.fit_transform(features_df)
    
    # Find optimal k using elbow method
    inertias = []
    K = range(2, 11)
    for k in K:
        km = KMeans(n_clusters=k, random_state=42, n_init=10)
        km.fit(X_scaled)
        inertias.append(km.inertia_)
    
    # Fit final model
    km_final = KMeans(n_clusters=n_clusters, random_state=42, n_init=10)
    features_df['cluster'] = km_final.fit_predict(X_scaled)
    
    # Profile each cluster
    cluster_profiles = features_df.groupby('cluster').mean()
    
    return features_df, cluster_profiles
```

---

## Pilot Structure (30 Days)

**Week 1:** Pull 12 months of transaction + behavioral data, clean and structure  
**Week 2:** Run RFM segmentation, identify top/at-risk segments  
**Week 3:** Build targeted campaign for one high-value segment  
**Week 4:** Measure campaign lift vs. control group

**Success metrics:**
- Email open rate by segment vs. unsegmented baseline (target: 30%+ lift for top segments)
- Conversion rate by segment (target: 2× lift for Champions vs. average)
- Revenue attributed to segmentation-driven campaigns
