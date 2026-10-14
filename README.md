# Predicting Mother-to-Child HIV Transmission (MTCT) Risk

A step-by-step data science notebook that profiles a PMTCT (Prevention of Mother-to-Child Transmission) cohort, identifies maternal and infant factors associated with infant HIV PCR outcome, and builds an interpretable classification model to flag infants at higher risk of transmission.

> **Notebook:** [`Stacey_MTCT_RISK_Step_by_Step.ipynb`](Stacey_MTCT_RISK_Step_by_Step.ipynb)
> **Platform:** Google Colab · **Language:** Python

---

## Project Goals
1. Describe the characteristics of mothers and infants in the dataset.
2. Identify important risk factors associated with MTCT.
3. Examine how maternal factors (viral load, CD4, ART adherence, etc.) relate to infant PCR outcome.
4. Build and compare models that handle a heavily imbalanced outcome.
5. Explain model behaviour in clinically meaningful terms using SHAP.


## Workflow
| Step | What it does |
**1. Load data** | Mounts Google Drive and loads the capstone Excel dataset with `pandas`. |
**2. Data quality profiling** | Missing values, duplicate rows, summary statistics, boxplots for outlier detection. |
**3. Cleaning** | Drops duplicates; median/mode imputation; range-based handling of implausible values (see below). |
**4. EDA** | Outcome distribution, maternal age distribution, age and viral load by PCR result, correlation matrix. |
**5. Feature significance testing** | t-tests (numeric) and chi-square tests (categorical) against PCR outcome. |
**6. Logistic regression** | Baseline model, then SMOTE and class-weight approaches to handle imbalance. |
**7. Model comparison** | Side-by-side plot of accuracy, positive-class recall and precision. |
**8. Explainability (SHAP)** | Summary and force plots showing which features push predictions toward Positive or Negative. |

### Data cleaning decisions

| Variable | Issue found | Treatment |
|----------|-------------|-----------|
| Maternal age | Implausible values (e.g. 8, 99) | Values outside 15–50 replaced with median |
| Maternal viral load | Placeholder values (e.g. 9,999,999) and zeros | Values > 100,000 or ≤ 0 replaced with median; log-transformed (`log1p`) for modelling |
| CD4 count | Negative and zero values | Replaced with median |
| Infant birth weight | Values outside 2–4 kg | Replaced with median (flagged as clinically important; range is adjustable) |
| Categorical fields | Inconsistent coding (`Good` / `GOOD` / `g`, feeding method abbreviations) | Standardised to consistent labels |

---

## Features Used in the Model

- Maternal age
- Maternal viral load (log-transformed)
- CD4 count
- Infant birth weight
- Gestational age at booking
- Gestational age at delivery
- ART adherence
- Infant feeding method

**Target:** infant PCR result (Positive = 1, Negative = 0)

## Results

The outcome is strongly imbalanced (far more Negative than Positive PCR results), so accuracy alone is misleading. Three configurations were compared on a stratified 70/30 train/test split (`random_state=42`):

| Model | Accuracy | Positive recall | Positive precision |
| Logistic regression (baseline) | 92.9% | 0.07 | 0.25 |
| + SMOTE (training set only) | 82.7% | 0.64 | 0.21 |
| + Class weights (`balanced`) | 70.2% | 0.86 | 0.16 |

**Takeaway**
- The baseline's high accuracy comes from predicting almost everything as Negative: it caught only 1 of 14 Positive cases.
- **SMOTE** gives a more balanced trade-off between sensitivity and false alarms.
- **Class weights** catch almost all Positive cases, which suits a *screening* scenario where missing a case is the costlier error, at the price of many false positives.

### SHAP findings

SHAP analysis of the model points in clinically expected directions:

- **Good ART adherence** is associated with lower predicted risk.
- **Higher maternal viral load** is associated with higher predicted risk.
- **Lower infant birth weight** and **shorter gestation** are associated with higher predicted risk.
- **Higher CD4 count** is associated with lower predicted risk.
- **Mixed feeding** is associated with higher predicted risk than replacement feeding.

---

## Limitations

- **Small number of positive cases.** The test set contains only 14 Positive infants, so recall and precision estimates are unstable and should not be over-interpreted.
- **Imputation before the train/test split.** Median/mode imputation and outlier replacement are applied to the full dataset before splitting, which can leak information from test data into training. A scikit-learn `Pipeline` would be cleaner.
- **Viral load handling.** Replacing all values above 100,000 copies/mL with the median removes values that may be clinically real (very high viral loads), and could weaken one of the most important predictors.
- **"Not applicable" ART adherence.** This category appears as a high-risk signal in SHAP but may reflect missing or unrecorded data rather than a true clinical group.
- **Association, not causation.** SHAP and regression coefficients describe patterns in this dataset; they do not establish causal risk factors.
- **Not a clinical tool.** This is an educational/research capstone and must not be used for clinical decision-making.

---

## Getting Started

### Requirements

```bash
pip install pandas numpy scipy scikit-learn imbalanced-learn seaborn matplotlib shap openpyxl
```
Data availability

The dataset contains health records used for a capstone programme and is **not included in this repository**. The notebook expects the following columns, among others:

`maternal_age_years`, `maternal_viral_load_copies_ml`, `cd4_count_cells_mm3`, `infant_birth_weight_kg`, `gestational_age_at_booking_weeks`, `gestational_age_at_delivery_weeks`, `art_adherence`, `infant_feeding_method`, `infant_pcr_result`

Tech Stack

`pandas` · `NumPy` · `SciPy` · `scikit-learn` · `imbalanced-learn (SMOTE)` · `seaborn` · `matplotlib` · `SHAP`

 Author:Stacey Nzighe Wachenje


## Acknowledgements

Dataset and capstone framework provided through the PU ENGAGE PMTCT capstone programme (Group 4).
