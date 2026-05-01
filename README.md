# Predicting High-Cost Medicare Patients Before They Become High-Cost

**ISE 503 Health Analytics | University of Southern California | Spring 2026**  
**Vansh Chanchlani & Sagarika Raju**

---

## Overview

5% of Medicare beneficiaries drive approximately 50% of total annual Medicare expenditures. Yet health systems and Accountable Care Organizations (ACOs) operating under value-based care contracts typically identify high-cost patients only *after* costs are incurred — leaving no window for proactive intervention.

This project builds and evaluates a binary classification pipeline that predicts which Medicare beneficiaries will fall in the **top 5% of annual spending in Year 2**, using only their Year 1 claims history. The output is a ranked risk-score list that ACO care management teams can use each January to prioritize proactive outreach before avoidable hospitalizations occur.

---

## Key Results

| Model | Features | Test AUC |
|---|---|---|
| Logistic Regression (Baseline) | 14 | 0.828 |
| XGBoost (Baseline) | 14 | 0.796 |
| Logistic Regression (Enriched) | 30 | 0.828 |
| **XGBoost (Enriched) ★** | **30** | **0.878** |

**Operational finding:** Flagging the top 10% of patients by predicted risk score captures **72% of all future high-cost patients**, who account for **56% of all projected Medicare spending**.

---

## Dataset

**Source:** [CMS Synthetic Medicare Public Use Files](https://data.cms.gov/collection/synthetic-medicare-enrollment-fee-for-service-claims-and-prescription-drug-event)  
A verified U.S. government source that mirrors the structure and statistical properties of real Medicare Research Identifiable Files.

**Files used:**
- Beneficiary Summary Files: 2022 (Year 1) and 2023 (Year 2)
- Inpatient Claims
- Outpatient Claims
- Carrier (Physician) Claims

**Final analytical dataset:** 3,339 patients · 30 features · 0 missing values  
**Label:** Top 5% of 2023 total Medicare spending (threshold: $85,798)  
**Class distribution:** 167 high-cost (5%) · 3,172 non-high-cost (95%)

---

## Features Engineered (30 total)

| Category | Features |
|---|---|
| Prior Spending | Total, Inpatient, Outpatient, Carrier payments |
| Utilization | Inpatient stays, days, outpatient visits, carrier claims |
| Admission Type | Emergency admits, elective admits, 30-day readmissions |
| Diagnosis Burden | Unique ICD code count, unique provider count |
| Chronic Disease Flags | Cardiovascular, diabetes, renal, cancer, mental health, respiratory, musculoskeletal, neurological |
| Demographics | Age, sex, race, coverage months |
| Spending Trajectory | Year-over-year spending change, growth rate |

---

## Methodology

```
CMS Data (2022/2023)
        ↓
Claims Linkage (BENE_ID join across 3 file types)
        ↓
Feature Engineering (30 features across 5 categories)
        ↓
Label Creation (Top 5% of Year 2 spending)
        ↓
Train/Test Split (70/30 stratified)
        ↓
Class Imbalance Handling (SMOTE for LR · scale_pos_weight for XGBoost)
        ↓
5-Fold Stratified Cross-Validation
        ↓
Model Selection (XGBoost Enriched · AUC 0.878)
        ↓
SHAP Interpretability + Risk Decile Analysis
```

---

## Model Performance

- **AUC-ROC:** 0.878 (vs. 0.828 baseline, exceeds Chechulin et al. 2014 benchmark of 0.78–0.82)
- **Average Precision:** 0.592 (vs. 0.050 random baseline — 12× better than chance)
- **Top Decile High-Cost Rate:** 36% (1 in 3 flagged patients actually became high-cost)
- **Capture Rate:** 72% of all future high-cost patients captured by flagging top 10%
- **Cost Coverage:** Top decile accounts for 56% of all Year 2 Medicare spending

---

## Top Predictors (SHAP Analysis)

| Rank | Feature | Mean |SHAP| |
|---|---|---|
| 1 | Total Prior Spending | 0.909 |
| 2 | Carrier Payments | 0.498 |
| 3 | Outpatient Visits | 0.486 |
| 4 | Age | 0.425 |
| 5 | Carrier Claims | 0.314 |
| 6 | Spending Growth Rate ★ | 0.305 |

★ Engineered feature — year-over-year spending trajectory

---

## Figures

| Figure | Description |
|---|---|
| Figure 1 | Year 2 Medicare spending distribution (raw + log scale) |
| Figure 2 | Year 1 spending by future high-cost status (13× gap) |
| Figure 3 | Demographic analysis by high-cost status |
| Figure 4 | Utilization patterns by high-cost status |
| Figure 5 | ROC curve — Logistic Regression baseline |
| Figure 6 | ROC comparison — LR vs XGBoost baseline |
| Figure 7 | SHAP beeswarm — Logistic Regression |
| Figure 8 | SHAP bar chart — Logistic Regression |
| Figure 9 | Risk decile analysis — XGBoost enriched |
| Figure 10 | ROC + Precision-Recall curves — all models |
| Figure 11 | SHAP beeswarm — XGBoost enriched |
| Figure 12 | SHAP bar chart — XGBoost enriched |
| Figure 13 | SHAP waterfall — highest-risk patient (predicted 0.979, actual: high-cost ✓) |
| Figure 14 | Calibration plot + cost concentration by risk decile |

---

## Repository Structure

```
├── Code/
│   └── medicare_project.ipynb    # Full analysis notebook
├── outputs/
│   ├── figure1_spending_distribution.png
│   ├── figure2_spending_by_label.png
│   ├── figure3_demographics.png
│   ├── figure4_utilization.png
│   ├── figure5_roc_curve.png
│   ├── figure6_roc_comparison.png
│   ├── figure7_shap_summary.png
│   ├── figure8_shap_bar.png
│   ├── figure9_risk_decile.png
│   ├── figure10_model_comparison.png
│   ├── figure11_shap_beeswarm.png
│   ├── figure12_shap_bar.png
│   ├── figure13_shap_waterfall.png
│   └── figure14_calibration_cost.png
└── .gitignore
```

---

## Libraries Used

```python
pandas · numpy · matplotlib · seaborn
scikit-learn · imbalanced-learn · xgboost · shap
```

---

## References

- Cohen, S. B., & Yu, W. (2012). The concentration and persistence in the level of health expenditures. *AHRQ Statistical Brief No. 354.*
- Chechulin, Y., et al. (2014). Predicting patients with high risk of becoming high-cost healthcare users in Ontario. *Healthcare Policy, 9(3).*
- Rose, S. (2016). A machine learning framework for plan payment normalization. *American Journal of Health Economics, 2(2).*
- Quan, H., et al. (2005). Coding algorithms for defining comorbidities in ICD-9-CM and ICD-10 administrative data. *Medical Care, 43(11).*
- Lundberg, S. M., & Lee, S.-I. (2017). A unified approach to interpreting model predictions. *NeurIPS 30.*
- CMS. (2023). Synthetic Medicare Public Use Files. https://data.cms.gov/

---

## Authors

**Sagarika Raju** · [LinkedIn](https://linkedin.com/in/sagarikaraju) · [GitHub](https://github.com/saga0302)  
**Vansh Chanchlani** · USC ISE 503 Health Analytics
