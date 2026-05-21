# 🏦 Credit Default Prediction

A comprehensive end-to-end data science project that combines **Exploratory Data Analysis (EDA)** with **Machine Learning** to predict loan default risk using real-world home credit application data. The goal is to help financial institutions identify high-risk applicants before loan disbursement, reducing credit losses.

---

## 📌 Table of Contents

- [Problem Statement](#problem-statement)
- [Dataset](#dataset)
- [Project Workflow](#project-workflow)
- [EDA Highlights](#eda-highlights)
- [Machine Learning Pipeline](#machine-learning-pipeline)
- [Model Results](#model-results)
- [Key Findings & Business Recommendations](#key-findings--business-recommendations)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)

---

## 🎯 Problem Statement

Financial institutions face significant losses from loan defaults. The challenge is to build a classification model that accurately predicts whether a loan applicant will **default (TARGET = 1)** or **repay (TARGET = 0)** — using information available at the time of application.

The dataset is **heavily imbalanced** (~8% default rate), making this a realistic and challenging problem that requires thoughtful handling of class imbalance.

---

## 📂 Dataset

Two datasets are used in this project:

| File | Description | Size |
|------|-------------|------|
| `application_data.csv` | Current loan application details (primary dataset) | ~307K rows |
| `previous_application.csv` | Historical loan applications for each client | ~1.67M rows |

**Key columns in `application_data.csv`:**

- `TARGET` — Binary target: 1 = defaulted, 0 = repaid
- `SK_ID_CURR` — Unique client identifier
- `CODE_GENDER`, `DAYS_BIRTH`, `NAME_EDUCATION_TYPE` — Applicant demographics
- `AMT_INCOME_TOTAL`, `AMT_CREDIT`, `AMT_ANNUITY`, `AMT_GOODS_PRICE` — Financial metrics
- `DAYS_EMPLOYED`, `OCCUPATION_TYPE`, `ORGANIZATION_TYPE` — Employment info
- `NAME_CONTRACT_TYPE`, `NAME_FAMILY_STATUS`, `NAME_HOUSING_TYPE` — Loan & lifestyle attributes

---

## 🔄 Project Workflow

```
Raw Data
   │
   ▼
Data Cleaning          ← Handle nulls, drop low-signal columns, fix formats
   │
   ▼
Univariate Analysis    ← Distributions, outlier detection, feature understanding
   │
   ▼
Bivariate/Multivariate ← Correlations with TARGET via bar plots, box plots, heatmaps
   │
   ▼
Dataset Merging        ← Inner join of application + previous application data
   │
   ▼
Feature Engineering    ← Label encoding, age grouping, day-to-year conversions
   │
   ▼
SMOTE Oversampling     ← Balance the imbalanced training set
   │
   ▼
Model Training         ← Logistic Regression, Decision Tree, Random Forest, XGBoost
   │
   ▼
Evaluation             ← Accuracy, Precision, Recall, F1, ROC-AUC, Confusion Matrix
   │
   ▼
Threshold Tuning       ← Optimize decision threshold for recall (business priority)
   │
   ▼
Feature Importance     ← Top predictors from Random Forest & XGBoost
   │
   ▼
Business Recommendations
```

---

## 📊 EDA Highlights

### Application Data Cleaning
- Dropped document flag columns and bureau inquiry columns (high null %, low signal)
- Removed columns with >45% missing values
- `AMT_ANNUITY` and `AMT_GOODS_PRICE` null rows dropped (<0.1% of data)
- `NAME_TYPE_SUITE` filled with mode (`Unaccompanied`)
- `ORGANIZATION_TYPE = XNA` mapped to `Pension`; corresponding `OCCUPATION_TYPE` set to `Pensioner`
- Remaining unknown occupations filled as `Unknown`
- Social circle columns filled with median
- Days columns (`DAYS_BIRTH`, `DAYS_EMPLOYED`, etc.) converted from negative days to positive years

### Outlier Handling
- Removed applicants with 10+ children (clear outliers)
- Capped `DAYS_EMPLOYED` at 60 years (1000+ year values were data errors)

### Key Univariate & Bivariate Insights

| Feature | Insight |
|---------|---------|
| **Age** | Youngest applicants (20–30) have the highest default rate |
| **Gender** | Males default more frequently than females |
| **Employment** | Shorter employment history → higher default risk |
| **Education** | Academic degree holders default least; low education → higher risk |
| **Loan Type** | Cash loans have a slightly higher default rate than revolving loans |
| **Family Status** | Single applicants and civil marriages show elevated default rates |
| **Car/Realty** | Owning assets correlates with lower default probability |
| **Region Rating** | Higher region rating (lower quality region) = higher default rate |

### Previous Application Data
- Cleaned analogously (dropped high-null and low-signal columns)
- Analyzed `NAME_CONTRACT_STATUS` distribution and its relationship to `AMT_APPLICATION`

### Merged Analysis
- Performed inner join on `SK_ID_CURR`
- Heatmaps of `CONTRACT_STATUS × FAMILY_STATUS → TARGET`
- Count plots of approved/refused/cancelled previous applications vs. default behavior

---

## 🤖 Machine Learning Pipeline

### 1. Feature Engineering
- Dropped `SK_ID_CURR` (ID column)
- Label encoded all categorical variables
- Created `AGE_GROUPS` bins: `20-30`, `30-40`, `40-50`, `50-60`, `60+`

### 2. Train-Test Split
- 80/20 stratified split to preserve class ratios

### 3. Class Imbalance — SMOTE
- Applied **SMOTE (Synthetic Minority Over-sampling Technique)** on the training set only
- Balanced the minority class (defaulters) to match the majority class

### 4. Scaling
- `StandardScaler` applied for Logistic Regression (tree-based models use raw features)

### 5. Models Trained

| Model | Key Hyperparameters |
|-------|---------------------|
| Logistic Regression | `max_iter=500` |
| Decision Tree | `max_depth=8` |
| Random Forest | `n_estimators=100`, `max_depth=10` |
| XGBoost | `n_estimators=100`, `max_depth=6`, `learning_rate=0.1` |

### 6. Threshold Tuning
- Default threshold of 0.5 was tuned for XGBoost
- Threshold of **0.3** chosen to maximize **Recall** — because missing a defaulter (False Negative) is costlier for the business than a false alarm

---

## 📈 Model Results

| Model | Accuracy | Recall (Default) | F1 (Default) | ROC-AUC |
|-------|:--------:|:----------------:|:------------:|:-------:|
| Logistic Regression | ~69% | Highest | Moderate | ~0.72 |
| Decision Tree | ~82% | Moderate | Moderate | ~0.68 |
| Random Forest | ~85% | Moderate | Good | ~0.78 |
| **XGBoost ⭐** | **~91%** | Moderate | **Best** | **~0.80** |

**XGBoost** was selected as the best model based on ROC-AUC and F1 score on the default class.

### Threshold Comparison (XGBoost)

| Threshold | Precision | Recall | F1 |
|-----------|-----------|--------|----|
| 0.5 (default) | Higher | Lower | Moderate |
| **0.3 (tuned)** | Lower | **Higher** | Comparable |
| 0.4 | Balanced | Balanced | Good |

---

## 💡 Key Findings & Business Recommendations

### Top Predictive Features (RF & XGBoost agreed)
1. **`DAYS_BIRTH` (Age)** — Strongest predictor; younger applicants default significantly more
2. **`AMT_CREDIT` / `AMT_GOODS_PRICE` / `AMT_ANNUITY`** — Loan burden relative to income
3. **`DAYS_EMPLOYED`** — Short employment tenure is a strong default signal
4. **`AMT_INCOME_TOTAL`** — Lower income → higher default probability
5. **`DAYS_REGISTRATION` / `DAYS_ID_PUBLISH`** — Document stability as a proxy for lifestyle stability

### Business Recommendations

1. **Lower decision threshold to 0.3** for high-risk segments to increase recall and catch more defaulters before disbursal
2. **Flag high-risk profiles**: Age 20–30 + short employment history + high credit-to-income ratio
3. **Risk-based pricing**: Apply higher interest rates for borderline applicants instead of flat rejection — retains potential revenue while pricing in default risk
4. **Fast-track low-risk segments**: Academic degree holders and applicants aged 60+ show the lowest default rates — streamline approvals for these groups
5. **Review rejection criteria**: A large share of previously refused applicants are non-defaulters, suggesting the current rejection policy may be over-conservative

---

## 🛠 Tech Stack

| Category | Libraries |
|----------|-----------|
| Data Manipulation | `pandas`, `numpy` |
| Visualization | `matplotlib`, `seaborn` |
| Machine Learning | `scikit-learn` |
| Boosting | `xgboost` |
| Imbalance Handling | `imbalanced-learn` (SMOTE) |
| Environment | Jupyter Notebook |
| Language | Python 3.x |

---

## 📁 Project Structure

```
Credit-Default-Prediction/
│
├── Credit_Default_Prediction.ipynb   # Main notebook (EDA + ML)
├── application_data.csv              # Primary dataset (loan applications)
├── previous_application.csv          # Secondary dataset (historical applications)
└── README.md                         # Project documentation
```

---

## ▶️ How to Run

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/Credit-Default-Prediction.git
   cd Credit-Default-Prediction
   ```

2. **Install dependencies**
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn xgboost imbalanced-learn jupyter
   ```

3. **Add the datasets**
   Place `application_data.csv` and `previous_application.csv` in the project root directory.
   *(Datasets available on [Kaggle – Home Credit Default Risk](https://www.kaggle.com/c/home-credit-default-risk/data))*

4. **Launch the notebook**
   ```bash
   jupyter notebook Credit_Default_Prediction.ipynb
   ```

5. **Run all cells** — the notebook is structured sequentially from data loading through EDA to ML modelling and final recommendations.

---

## 🤝 Connect

Feel free to reach out or raise an issue if you have questions, feedback, or suggestions for improving the model!

