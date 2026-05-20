# 📱 Telecom Customer Churn Prediction

> Predicting high-value customer churn for a telecom provider using logistic regression (with RFE + statsmodels) and PCA-based dimensionality reduction — applied to 100,000 prepaid customers across 226 usage features over 4 months.

---

## 📌 Problem Statement

Customer churn is one of the most costly problems in the telecom industry. This project focuses specifically on **high-value customers** (top 30% by recharge amount) — who generate the majority of revenue. The goal is to predict whether a customer will churn in month 9 based on their behaviour in months 6, 7, and 8.

**Churn Definition:** A customer is labelled as churned if their total incoming calls, outgoing calls, 2G volume, and 3G volume in month 9 are all zero.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | Telecom prepaid customer data |
| Original Size | 99,999 customers × 226 features |
| After High-Value Filter | 30,001 customers × 141 features |
| Target | `churn` (1 = churned, 0 = retained) |
| Class Distribution | 91.9% Not Churned · 8.1% Churned |
| Time Period | Months 6, 7, 8 (features) → Month 9 (churn label) |

---

## 🧠 Approach

### 1. Data Understanding & Cleaning
- Identified **226 features** across 4 months of usage data
- Dropped **16 columns** with a single unique value (zero variance)
- Converted 8 date columns to datetime format
- Identified **40 columns** with >50% missing values

### 2. Handling Missing Values
- **Data recharge columns** (`total_rech_data_*`, `max_rech_data_*`): imputed with 0 where no recharge date exists — indicating no data recharge activity
- **Average recharge amount** (`av_rech_amt_data_*`): imputed with 0 where total data recharges = 0
- Dropped `count_rech_2g/3g`, `arpu_3g/2g`, `fb_user`, `night_pck_user` columns (high missing values or redundant)
- Dropped all date columns after use
- Final missing values handled via **KNN Imputer** (k=3) after MinMax scaling

### 3. High-Value Customer Filtering
- Computed average recharge amount across months 6 and 7 (voice + data)
- Filtered to **top 30%** by average recharge — the high-value segment
- Cut-off threshold: 70th percentile of `avg_rech_amt_6_7`

### 4. Churn Labelling
- Customers with zero usage across all 4 churn-phase indicators in month 9 are labelled as churned
- All month-9 columns dropped after labelling (to prevent data leakage)

### 5. Feature Engineering
- `tenure` — derived from `aon` (age on network in months)
- `tenure_range` — binned into 5 categories: 0–6M, 6–12M, 1–2Yrs, 2–5Yrs, 5Yrs+
- `avg_arpu_6_7` — average revenue per user across months 6 and 7
- `overall_rech_amt_6/7` — combined voice + data recharge amounts
- Dummy variables created for `tenure_range`, `total_rech_data_group_8`, `total_rech_num_group_8`

### 6. Class Imbalance
- Applied **SMOTE** (Synthetic Minority Oversampling Technique) on training data
- Balanced training set: 38,576 samples (19,288 churned + 19,288 not churned)

---

## 🤖 Models

### Model 1 — Logistic Regression with RFE + Statsmodels

| Step | Detail |
|------|--------|
| Feature Selection | RFE with Logistic Regression → top 20 features |
| Refinement | VIF analysis + manual removal of high-multicollinearity features |
| Final Features | 19 features |
| Framework | `statsmodels` GLM (Binomial family, Logit link) |
| Optimal Threshold | 0.54 (tuned by evaluating thresholds 0.50–0.59) |

**Top predictive features selected by RFE:**

| Feature | Direction |
|---------|-----------|
| `total_ic_mou_8` | Strong negative (↑ usage → ↓ churn) |
| `avg_arpu_6_7` | Strong positive (↑ ARPU → ↑ churn risk in this segment) |
| `last_day_rch_amt_8` | Negative (recharge activity → ↓ churn) |
| `std_ic_mou_6` | Positive |
| `total_og_mou_8` | Negative |
| `spl_ic_mou_8` | Negative |
| `aug_vbc_3g` | Negative |
| `monthly_2g_8` | Negative |

**Results:**

| Set | Accuracy | Notes |
|-----|----------|-------|
| Train (SMOTE) | 83.6% | Threshold = 0.54 |
| Test | 83.4% | Threshold = 0.54 |

**Confusion Matrix (Test Set):**
```
              Predicted 0   Predicted 1
Actual 0         6918          1354
Actual 1          143           586
```

---

### Model 2 — Logistic Regression with PCA

| Step | Detail |
|------|--------|
| Scaling | MinMaxScaler |
| Imbalance | SMOTE (38,576 balanced samples) |
| Dimensionality Reduction | PCA — full (148 components) then reduced to 15 |
| Model | Logistic Regression on PCA components |

**Scree Plot Analysis:** Explained variance flattens after ~15 components → selected `n_components=15`

**Results:**

| PCA Components | Accuracy |
|---------------|----------|
| Full (148) | 81.8% |
| Reduced (15) | 76.0% |

**Confusion Matrix (15 PCA components):**
```
              Predicted 0   Predicted 1
Actual 0         6302          1970
Actual 1          187           542
```

---

## 📈 Model Comparison

| Model | Test Accuracy | Key Advantage |
|-------|--------------|---------------|
| Logistic Regression (RFE) | **83.4%** | Interpretable — shows which features drive churn |
| Logistic Regression (Full PCA) | 81.8% | Handles multicollinearity |
| Logistic Regression (15 PCA) | 76.0% | Most compact — 15 components only |

**Best Model: Logistic Regression with RFE** — highest accuracy and interpretable coefficients with VIF-validated features.

---

## 📊 Key Visualisations

- Churn rate by tenure range (bar plot)
- Average ARPU distribution (distplot)
- Total recharge vs ARPU scatter plot (month 8)
- Tenure vs churn box plot
- Total recharge data and count distributions by churn (count plots)
- ROC Curve — Train and Test sets
- PCA scree plot (explained variance)
- PCA cumulative explained variance curve

---

## 🛠️ Tools & Libraries

| Category | Tools |
|----------|-------|
| **Language** | Python 3 |
| **Data** | Pandas · NumPy |
| **ML** | Scikit-learn (LogisticRegression, RFE, PCA, SMOTE, MinMaxScaler, KNNImputer) |
| **Statistics** | Statsmodels (GLM Binomial, VIF) |
| **Imbalance** | imbalanced-learn (SMOTE) |
| **Visualization** | Matplotlib · Seaborn |
| **Environment** | Jupyter Notebook |

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd Telecom-Churn-Prediction
pip install -r requirements.txt
jupyter notebook
```

**Required packages:**
```
pandas numpy scikit-learn statsmodels imbalanced-learn matplotlib seaborn
```

---

## 📁 File Structure

```
Telecom-Churn-Prediction/
├── telecom_churn.ipynb         # Main notebook
├── telecom_churn_data.csv      # Dataset
└── README.md                   # This file
```

---

## 🔑 Key Takeaways

- **Churn is rare but costly** — only 8.1% of high-value customers churned; SMOTE was essential to prevent the model from ignoring the minority class
- **Month 8 usage is the strongest signal** — features from the action phase (`_8` columns) dominate RFE selection, confirming early behavioural decline predicts churn
- **Total incoming call minutes in month 8 (`total_ic_mou_8`)** is the single strongest negative predictor — customers still receiving calls are unlikely to churn
- **VIF analysis** revealed `arpu_6` has high multicollinearity (VIF=59) yet was retained due to statistical significance
- **PCA at 15 components** explains most variance but loses interpretability vs the RFE approach
- **Shorter-tenure customers** (0–12 months) churn at a higher rate than long-term subscribers

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
