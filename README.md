# Mobile Payment Fraud Detection & Risk Scoring

> End-to-end data analyst project using Python, MySQL, Power BI, and the PaySim dataset.

---

# The Problem

MobiPay's automated fraud flag (`isFlaggedFraud`) caught **16 out of 8,213** confirmed fraud cases in 2024.

That is a **0.19% recall rate**.

While ₹12.06 billion moved through undetected fraud, the system designed to stop it was flagging one transaction per 514 fraudulent ones.

This project rebuilds the detection framework from scratch using behavioral signal engineering, producing a tiered risk model that achieves:

- **73× improvement in recall**
- **64.5% precision** in the Critical risk tier

High enough to justify automated transaction holds.

---

# Key Findings

| Finding | Detail |
|---|---|
| Fraud is exclusive to 2 transaction types | `TRANSFER` (3.73%) and `CASH_OUT` (0.91%). `PAYMENT`, `CASH_IN`, and `DEBIT` show **zero fraud** across 718K transactions |
| 2–5 AM danger zone | Fraud rate at 3 AM reached **57.56%** |
| Amount sweet spot | ₹10L–1Cr transactions show **9.69% fraud rate** |
| Round amounts are suspicious | Amounts divisible by ₹1,000 show **30.5% fraud rate** |
| Weekly escalation pattern | Fraud rate increased from 0.49% (Week 1) to **5.75% (Week 5)** |

---

# Risk Model Results

| Risk Tier | Transactions | Fraud Cases | Fraud Rate | Avg Amount |
|---|---:|---:|---:|---:|
| 🔴 Critical | 344 | 222 | **64.53%** | ₹1.00 Cr |
| 🟠 High | 1,938 | 929 | **47.94%** | ₹97.6 L |
| 🟡 Medium | 81,679 | 5,592 | **6.85%** | ₹3.6 L |
| 🟢 Low | 182,661 | 1,449 | **0.79%** | ₹3.1 L |

Fraud rate is **strictly monotonically increasing**:

`Critical > High > Medium > Low`

---

# System Comparison

| System | Flags Raised | True Positives | Precision | Recall |
|---|---:|---:|---:|---:|
| Legacy (`isFlaggedFraud`) | 16 | 16 | 100.00% | 0.19% |
| Our Model — Critical only | 344 | 222 | 64.53% | 2.70% |
| Our Model — High + Critical | 2,282 | 1,151 | 50.44% | **14.01%** |

## Result

**73× recall improvement** over the existing system.

---

# Project Structure

```text
paysim-fraud-detection/
│
├── data/
│   ├── PS_20174392719_1491204439457_log.csv
│   └── paysim_enriched.csv
│
├── paysim_analysis.py
├── paysim_queries.sql
│
├── dashboard/
│   └── upi_fraud_analysis.pbix
│
├── paysim_fraud_report.pdf
└── README.md
```

---

# Pipeline

```text
Raw CSV (6.36M rows)
      │
      ▼
[Python — paysim_analysis.py]

• Stratified sampling
  - Keep all fraud rows
  - Keep 20% non-fraud rows
  - Final dataset: 1.28M rows

• Time Features
  - hour_of_day
  - day_of_sim
  - week_of_sim
  - is_offhours

• Signal Features
  - is_round_amount
  - dest_was_empty
  - orig_zeroed_out
  - is_amount_anomaly

• Risk Scoring
  - Data-derived risk weights
  - Tier assignment:
    Critical / High / Medium / Low

      │
      ▼

paysim_enriched.csv

      ├── MySQL Analysis
      │   ├── Q1: System failure analysis
      │   ├── Q2: Fraud by transaction type
      │   ├── Q3: Weekly GMV + fraud trend
      │   ├── Q4: Hourly fraud rate
      │   ├── Q5: Feature signal strength
      │   ├── Q6: Risk tier validation
      │   ├── Q7: Repeat offender detection
      │   ├── Q8: Amount segmentation
      │   └── Q9: Top 50 high-risk transactions
      │
      └── Power BI Dashboard
          ├── Page 1: The Problem
          ├── Page 2: The Discovery
          └── Page 3: The Solution
```

---

# Feature Engineering

All 5 signals were validated empirically.

Weights were derived from observed fraud rates rather than hand-tuned constants.

```python
Signal                Fraud Rate    Weight
-------------------------------------------
is_amount_anomaly      50.44%        50.9
is_round_amount        30.53%        30.8
is_offhours             5.01%         5.1
orig_zeroed_out         2.58%         2.6
dest_was_empty          0.98%         1.0
```

## Anomaly Detection Logic

Uses:

- **Type-stratified IQR**
- Applied on `log(amount)`
- 2.5× multiplier threshold

This performs significantly better than raw Z-score methods on right-skewed financial distributions.

---

# Actionable Recommendations

## 🔴 Immediate — Automated Hold

Flag all `TRANSFER` and `CASH_OUT` transactions between:

- 02:00–05:00
- Amount > ₹10L
- First-time sender-receiver pairs

Critical tier precision reaches **64.5%**, making automated blocking defensible.

---

## 🟠 Short-Term — Step-Up Authentication

Require OTP re-verification for:

- Round amounts
- Amount anomalies
- Off-hours activity

This captures the bulk of detectable fraud value.

---

## 🟡 Medium-Term — Velocity Features

Add:

- Transactions per account per hour
- Rapid recipient creation
- Burst transaction patterns

The remaining Low-tier fraud likely shares velocity-based behavior.

---

## 🔵 Long-Term — ML Extension

Potential next step:

- XGBoost
- SMOTE balancing
- Feature-engineered dataset

Target metrics:

- Recall > 70%
- Precision > 50%

---

# Tech Stack

| Tool | Purpose |
|---|---|
| Python (`pandas`, `numpy`) | Feature engineering and risk scoring |
| MySQL 8.0 | Analytical queries on large datasets |
| Power BI | Interactive dashboard |
| PaySim Dataset | Mobile money fraud simulation |

---

# Dataset

## PaySim — Synthetic Financial Dataset for Fraud Detection

Source: https://www.kaggle.com/datasets/ealaxi/paysim1

Reference:

> Lopez-Rojas, E., Elmir, A., & Axelsson, S. (2016).  
> *PaySim: A financial mobile money simulator for fraud detection.*

### Dataset Summary

- 6,362,620 rows
- 11 columns
- 0 null values
- 30 simulated days of mobile money transactions
- Fraud exists only in:
  - `TRANSFER`
  - `CASH_OUT`

---

# Limitations

- 14% recall still leaves substantial fraud undetected
- PaySim is synthetic and requires recalibration for production systems
- Risk weights should be retrained periodically as fraud behavior evolves

---

# Author

Built as part of a Data Analyst portfolio project targeting fintech and product-focused companies.

### Tools Used

- Python
- MySQL
- Power BI

### Focus Areas

- Fraud Detection
- Risk Scoring
- Financial Analytics
