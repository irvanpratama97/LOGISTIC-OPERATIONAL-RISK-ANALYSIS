# LOGISTIC-OPERATIONAL-RISK-ANALYSIS
Quantifying How Fleet Overutilization Drives Mechanical Failure, Delivery Delays, and Revenue Loss

## Skills Demonstrated

- Data Cleaning
- Exploratory Data Analysis (EDA)
- Pearson Correlation Analysis
- Logistic Regression
- Linear Regression (OLS)
- Counterfactual Simulation
- Business Impact Analysis
- Data Visualization (Tableau)
- SQL Query Optimization (CTE)
- Business Recommendation Development

# Operational Risk Analysis: Fleet Overutilization, Delivery Delays, and Revenue Impact

## Overview

This project investigates how fleet overutilization contributes to mechanical failures, delivery delays, and ultimately revenue loss in a logistics operation.

Using SQL, Python, and Tableau, the analysis follows a complete analytical workflow—from descriptive analytics and exploratory analysis to statistical modeling and business impact simulation—to identify operational risks and estimate the financial value of mitigating them.

> **Objective:** Quantify the operational and financial impact of fleet overutilization and provide actionable recommendations for improving logistics performance.

---

## Business Problem

A logistics company experienced a high delivery delay rate but lacked visibility into the operational factors driving those delays.

The project focuses on answering the following business questions:

- Does delivery delay affect customer transaction value?
- Does fleet overutilization increase the likelihood of mechanical failure?
- How much additional delay is caused by mechanical failures?
- What financial impact could be achieved by reducing fleet overutilization?

---

## Dataset

**Observations:** 1,000 deliveries

### Key Variables

- Asset_ID
- Logistics_Delay
- Delay_Time
- User_Transaction_Amount
- is_overutilized
- is_underutilized
- is_mechanical_failure

---

# Analytical Workflow

## Phase 1 — Descriptive Analytics (Tableau)

Interactive dashboard used to identify operational bottlenecks.

### Key Findings

- Total Deliveries: **1,000**
- Delivery Delay Rate: **56.6%**
- Mechanical Failures: **133**
- Overutilized Assets: **238**
- Underutilized Assets: **267**
- Overheated Assets: **299**

The dashboard highlighted **Mechanical Failure** as the largest controllable operational issue.

---

## Phase 2 — Exploratory Analysis (MySQL)

Performed asset-level Pearson correlation analysis using Common Table Expressions (CTEs).

### Objective

Measure the relationship between delivery delays and customer transaction values.

### Result

Strong negative correlations were observed across fleet assets.
This indicates that longer delivery delays are associated with lower customer transaction amounts.

---

## Phase 3 — Statistical Modeling (Python)

To avoid biased conclusions, the analysis used a **Two-Part Statistical Model** instead of relying on a single regression.

### Why?

The dataset contains:

- Zero-inflated delay distribution
- Complete separation in logistic regression
- Binary and continuous response variables

Therefore, two separate models were applied.

### Model 1

**Logistic Regression**

```
is_mechanical_failure ~ is_overutilized
```

Purpose:

Estimate the probability of mechanical failure.

Key Result

- Odds Ratio = **1.2218**
- Overutilized assets have **22.2% higher odds** of experiencing mechanical failure.

---

### Model 2

**OLS Regression**

```
Delay_Time ~ is_mechanical_failure
```

Purpose:

Estimate additional delay caused by mechanical failure.

Key Result

Mechanical failures increase delivery delay by approximately

```
0.70 hours
```

---

### Model 3

**Revenue Impact Regression**

```
User_Transaction_Amount ~ Delay_Time
```

Purpose:

Estimate financial loss per hour of delivery delay.

Key Result

Every additional hour of delay reduces customer transaction value by

```
Rp 331,410
```

---

# Counterfactual Simulation

A business simulation was conducted to estimate the impact of eliminating fleet overutilization while preserving the natural baseline failure risk.

## Results

Prevented Mechanical Failures

```
5.7 cases
```

Hours Saved

```
198.46 hours
```

Estimated Revenue Recovered

```
Rp 65,772,225
```

---

# Business Recommendations

- Balance workload across fleet assets to reduce overutilization.
- Implement predictive maintenance scheduling for high-risk vehicles.
- Monitor fleet utilization continuously using operational dashboards.
- Track delay-related financial losses as an operational KPI.

---

# Technologies

- Python
    - Pandas
    - NumPy
    - Statsmodels
    - SciPy
- MySQL
- Tableau
- Jupyter Notebook

---

# Repository Structure

```
├── data/
│
├── sql/
│   ├── correlation_analysis.sql
│
├── notebooks/
│   ├── operational_analysis.ipynb
│
├── dashboard/
│   ├── tableau_dashboard.twb
│
├── images/
│
└── README.md
```

---

# Python Code
```python
import pandas as pd
import numpy as np
import statsmodels.api as sm
import statsmodels.formula.api as smf

# --- DATA PREPARATION ---
df['is_delay'] = np.where(df['Delay_Time'] > 0, 1, 0)
df['is_overutilized'] = np.where(df['Asset_Utilization_Status'] == 'Overutilized', 1, 0)
df['is_mechanical_failure'] = np.where(df['Logistics_Delay_Reason'] == 'Mechanical Failure', 1, 0)


# --- CAUSALITY RELATIONSHIP ANALYSIS ---

# A
model_penyebab = smf.logit('is_mechanical_failure ~ is_overutilized', data=df).fit()
print("--- ANALISYS 1: Overutilized Impact to Mechanical Failure ---")
print(model_penyebab.summary())
print("Odds Ratio:\n", np.exp(model_penyebab.params))
print("\n" + "="*50 + "\n")

# B
df_delayed = df[df['Delay_Time'] > 0]
model_durasi = smf.ols('Delay_Time ~ is_mechanical_failure', data=df_delayed).fit()
print("--- ANALISYS 2: Mechanical Failure Impact to Delay Time ---")
print(model_durasi.summary())
print("\n" + "="*50 + "\n")


# --- TRANSACTION REGRESSION VS DELAY TIME ---
model_finance = smf.ols('User_Transaction_Amount ~ Delay_Time', data=df).fit()
beta_delay = model_finance.params['Delay_Time']
print("--- ANALISYS 3: Delay Time Impact to Purchase Amount ---")
print(model_finance.summary())
print(f"Amount of Hourly Loss: Rp {abs(beta_delay):,.2f}")
print("\n" + "="*50 + "\n")

# --- IMPACT SIMULATION (Overutilized Asset = 0) ---

avg_hours_mf = df_delayed[df_delayed['is_mechanical_failure'] == 1]['Delay_Time'].mean()

p_mf_normal = df[df['is_overutilized'] == 0]['is_mechanical_failure'].mean()

estimated_new_mf = len(df) * p_mf_normal
current_mf = df['is_mechanical_failure'].sum()

delta_mf = max(0, current_mf - estimated_new_mf)

hours_saved = delta_mf * avg_hours_mf

revenue_recovered = hours_saved * (-beta_delay)

print("--- IMPACT SIMULATION RESULT (Overutilized = 0) ---")
print(f"Estimated Reduced Mechanical Failure : {delta_mf:.1f} cases")
print(f"Estimated Hours Saved    : {hours_saved:.2f} hours")
print(f"Estimated Revenue Recovered: Rp {revenue_recovered:,.2f}")
