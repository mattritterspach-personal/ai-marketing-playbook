# Module 5: Recommended AI Marketing Stack

## Overview

A curated, opinionated tool stack for the full AI × Marketing system. Organized by function with vendor comparisons, build-vs-buy guidance, and integration notes.

---

## Stack by Function

### Content Production

| Category | Recommended | Alternative | Notes |
|----------|------------|-------------|-------|
| LLM / Writing | Claude (Anthropic) | GPT-4o (OpenAI) | Claude excels at long-form; GPT-4o better for structured outputs |
| SEO optimization | Surfer SEO | Clearscope, MarketMuse | Surfer has best editor integration |
| Content workflow | Notion AI | Jasper | Notion better if team already uses it |
| Image generation | Midjourney | DALL-E 3, Firefly | Midjourney for quality; Firefly for commercial safety |

**Monthly cost estimate:** $200–$500/mo for a team of 3–5

### Attribution

| Category | Recommended | Alternative | Notes |
|----------|------------|-------------|-------|
| Multi-touch SaaS | Northbeam | Rockerbox, Triple Whale | Northbeam best for B2B/longer cycles |
| Custom modeling | Python (Shapley) | R | Open source, build once |
| Experimentation | Statsig | LaunchDarkly, Optimizely | Statsig has best built-in analytics |
| Data warehouse | BigQuery | Snowflake | BigQuery cheapest for <1TB/mo |

**Monthly cost estimate:** $1,500–$5,000/mo (Northbeam pricing based on spend)

### Segmentation

| Category | Recommended | Alternative | Notes |
|----------|------------|-------------|-------|
| CDP | Segment | Rudderstack | Segment more mature; Rudderstack is cheaper/OSS |
| CRM | HubSpot | Salesforce | HubSpot better for SMB/mid-market |
| ML platform | Python (scikit-learn) | SageMaker | DIY for cost control; SageMaker for scale |
| Email ESP | Klaviyo | Iterable, Braze | Klaviyo best for e-commerce; Braze for enterprise |

**Monthly cost estimate:** $500–$3,000/mo depending on contact volume

### Forecasting

| Category | Recommended | Alternative | Notes |
|----------|------------|-------------|-------|
| Time-series | Prophet (Meta) | statsmodels, Darts | Prophet easiest; Darts most powerful |
| BI / Dashboarding | Looker | Metabase, Tableau | Metabase for self-hosted; Looker for governance |
| Orchestration | dbt + Airflow | Prefect, Dagster | dbt for SQL transforms; Airflow for scheduling |
| Revenue ops | Clari | Gong Forecast | Clari best pipeline AI |

**Monthly cost estimate:** $2,000–$8,000/mo depending on tools

---

## Build vs. Buy Decision Framework

**Build when:**
- Your use case is unique to your business model
- You need full control over the model and output
- You have a data scientist or ML engineer on staff
- The vendor solution doesn't integrate with your stack

**Buy when:**
- A SaaS solution covers 80%+ of your needs
- Time-to-value matters more than cost optimization
- The category is mature and competitive (prices drop over time)
- Your team lacks ML expertise

**Hybrid (most common):**
- Use SaaS for data collection and pipeline (CDP, ESP, CRM)
- Build custom models in Python/SQL on top of that data
- Use BI tools for visualization, not computation

---

## Minimal Viable Stack (< $1,000/mo)

For early-stage or budget-constrained teams:

1. **Claude API** — Content AI ($20–$100/mo depending on usage)
2. **Segment free tier** — Customer data (free up to 1,000 MTUs)
3. **Metabase OSS** — Dashboards (free, self-hosted)
4. **Python + Prophet** — Forecasting (free, open source)
5. **HubSpot Starter** — CRM + email ($45/mo)
6. **GA4** — Attribution baseline (free)

Total: ~$200–$300/mo before headcount
