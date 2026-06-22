# 🗽 NYC Borough Prediction & Property Price Prediction

> A two-part machine learning project on 84,548 NYC property sales transactions — first predicting which of NYC's five boroughs a property belongs to (multiclass classification), then predicting its sale price (regression). Both sections use 6 models each: Logistic Regression / Linear Regression, Decision Tree, SVC / SVR, Random Forest, Gradient Boosting, and Neural Networks.

---

## 📌 Part 1 — Borough Prediction (Classification)

### Problem Statement

New York City is a rapidly growing urban center with significant opportunities for business and residential development. This section builds a multiclass classification model to predict the **borough** of a property based on features like tax class, sale price, building age, unit counts, and square footage.

**Target classes:** Brooklyn · Queens · Bronx · Staten Island · Manhattan

The model aims to support stakeholders in making informed and efficient decisions for business or residential planning in NYC's complex real estate market.

---

### 📊 Dataset

| Detail | Info |
|--------|------|
| Source | NYC Property Sales (Kaggle) |
| Size | **84,548 rows × 22 columns** |
| Each row | One real estate sale transaction in NYC |
| Target | `Borough` (5 classes) |
| Null rows | 39,215 rows with missing values (most had >2 columns missing — all deleted) |
| Duplicates | None |

**Class Distribution (imbalanced):**

| Borough | Share |
|---------|-------|
| Brooklyn | 48% |
| Queens | 24% |
| Bronx | 13% |
| Staten Island | 11% |
| Manhattan | 2% |

---

### 🧠 Approach

**Exploratory Data Analysis**
- All numerical features are **right-skewed** in distribution
- Residential units significantly outnumber commercial units relative to total units
- Borough target variable is **heavily imbalanced** — Brooklyn dominates at 48%

**Feature Engineering & Preprocessing**
- `Year Built` → converted to **Building Age** by subtracting from 2025
- Removed address and zip code columns to prevent **horizontal data leakage**
- Removed redundant columns and high-cardinality categorical features
- **Train / Validation / Test Split:** 70% · 15% · 15%
- Numerical pipeline: Median imputation → Log transformation → Standard Scaler
- Categorical pipeline: Most-frequent imputation → One-Hot Encoding

**Class Imbalance Handling**
- Separate Imblearn pipeline for Logistic Regression: down-sampling then SMOTE oversampling
- After balancing: all 5 classes have **2,700 rows** each
- All other models trained on the **unbalanced** dataset

---

### 🤖 Models — Borough Classification

#### 1. Logistic Regression (Balanced & Unbalanced)

| Setting | Detail |
|---------|--------|
| CV | GridSearchCV · 5 folds |
| Parameter | `C` — 10 log-spaced values between 10⁻³ and 10² |
| Best C | **27.82** |

- Unbalanced model **outperformed** the balanced model — a surprising finding

| Model | F1 (macro) | Accuracy |
|-------|-----------|---------|
| Logistic Regression (Balanced) | 0.47 | 0.53 |
| Logistic Regression (Unbalanced) | **0.48** | **0.63** |

---

#### 2. Decision Tree Classifier

| Setting | Detail |
|---------|--------|
| CV | GridSearchCV · 5 folds · `max_depth` 1–15 |
| Best max_depth | **13** |

**Test Results:** Accuracy = **0.72** · F1 (macro) = **0.68**

---

#### 3. Support Vector Classifier (SVC)

| Setting | Detail |
|---------|--------|
| Dataset | Resampled to **1,000** records (SVC too slow on full data) |
| Parameters | `C` · `Gamma` · `Kernel` (RBF, Poly) — 1,000 total fits |
| Best parameters | C = **51.41** · Gamma = **5.0** · Kernel = **RBF** |

**Test Results:** Accuracy = **0.63** · F1 (macro) = **0.53**

---

#### 4. Random Forest ⭐ Best Classification Model

| Setting | Detail |
|---------|--------|
| CV | GridSearchCV · `max_depth` 10–30 · `n_estimators` 100–1000 |
| Total fits | **225** |
| Best parameters | max_depth = **20** · n_estimators = **700** |

**Top Features:** `Sale Price` and `Land Square Feet` contribute most.

**Test Results:** Accuracy = **0.76** · F1 (macro) = **0.72** ← Best overall

---

#### 5. Gradient Boosting

| Setting | Detail |
|---------|--------|
| Parameters | depth = 4 · max estimators = 500 · early stopping patience = 10 |
| Trees used | **14** (early stopping) |

**Test Results:** Accuracy = **0.73** · F1 (macro) = **0.67**

---

#### 6. Neural Network

| Setting | Detail |
|---------|--------|
| Architectures tested | 5 |
| Best | 5 hidden layers — 100 → 10 neurons (tapering) |
| Callbacks | EarlyStopping(patience=10) |

- Validation loss decrease less evident than training loss — signs of mild overfitting

**Test Results:** Accuracy = **0.74** · F1 (macro) = **0.68**

---

### 📈 Borough Classification — Full Model Comparison

| Model | Train F1 | Test F1 | Test Accuracy |
|-------|---------|---------|--------------|
| Logistic Regression (Unbalanced) | 0.48 | 0.48 | 0.63 |
| Decision Tree | 0.67 | 0.68 | 0.72 |
| SVC | 0.55 | 0.53 | 0.63 |
| **Random Forest** | **0.75** | **0.72** | **0.76** |
| Gradient Boosting | 0.71 | 0.67 | 0.73 |
| Neural Network | 0.65 | 0.68 | 0.72 |

---

### 🏆 Borough Prediction — Conclusion

**Random Forest** is the best classification model — highest test F1 macro of **0.72** and accuracy of **76%**.

Key findings:
- Simpler models (Logistic Regression, SVC) struggled with the 5-class imbalanced problem
- Ensemble methods significantly outperformed linear and kernel-based models
- The balanced dataset did **not** improve Logistic Regression — the unbalanced version performed better, showing that balancing alone does not guarantee improvement for all model types
- **Manhattan (only 2% of data)** was the hardest class to predict — severe underrepresentation leads to poor recall

---

---

## 📌 Part 2 — Property Price Prediction (Regression)

### Problem Statement

Building on the same dataset and preprocessing pipeline used for borough classification, this section reframes the problem as a **regression task** — predicting the actual sale price of a NYC property. BOROUGH now becomes an input feature (instead of the target), and SALE PRICE becomes the target.

---

### 📊 Dataset

| Detail | Info |
|--------|------|
| Source | Same NYC Property Sales dataset |
| Raw rows | 84,548 |
| After cleaning | 45,333 |
| After price filter ($10K–$10M) | **34,636** |
| Features | 9 (BOROUGH · TAX CLASS AT PRESENT · RESIDENTIAL UNITS · COMMERCIAL UNITS · TOTAL UNITS · LAND SQUARE FEET · GROSS SQUARE FEET · BUILDING AGE · TAX CLASS AT TIME OF SALE) |
| Target | `SALE PRICE` (continuous) |

**Why filter to $10,000–$10,000,000?**
- **22.4% of records had sale price < $10,000** — these are non-arm's-length transfers (family deeds, corrections, $0 administrative transactions) that do not reflect true market value
- **1.2% of records exceeded $10M** — bulk portfolio or institutional sales that skew the distribution
- Filtering retains **76.4% of the cleaned data** (34,636 rows) representing genuine market-rate property transactions

**Sale Price Distribution (filtered):**
- Median: **$610,000**
- Mean: **$884,000**
- Range: $10,000 → $10,000,000

---

### 🧠 Approach

**Preprocessing Pipeline (reused from Part 1, adapted for regression)**
- Same feature engineering: `Year Built` → `BUILDING AGE`
- Same leakage prevention: address, zip code, high-cardinality columns dropped
- **Train / Validation / Test Split:** 70% · 15% · 15%
- Two pipeline variants:
  - **Scaled** (Median imputation → Log1p → StandardScaler): used for Ridge and SVR
  - **Unscaled** (Median imputation only): used for tree-based models and NN
- Categorical: One-Hot Encoding (drop first)

---

### 🤖 Models — Property Price Regression

#### 1. Linear Regression (Ridge)

| Setting | Detail |
|---------|--------|
| CV | GridSearchCV · 5 folds |
| Parameter | `alpha` — 10 log-spaced values between 10⁻³ and 10³ |
| Best alpha | **0.464** |
| Best CV R² | **0.40** |

**Test Results:** R² = **0.36** · RMSE = **$818,722** · MAE = **$412,614**

---

#### 2. Decision Tree Regressor

| Setting | Detail |
|---------|--------|
| CV | GridSearchCV · 5 folds |
| Parameter | `max_depth` — values 1 to 14 |
| Best max_depth | **6** |
| Best CV R² | **0.52** |

**Test Results:** R² = **0.51** · RMSE = **$718,495** · MAE = **$371,546**

---

#### 3. Support Vector Regressor (SVR)

| Setting | Detail |
|---------|--------|
| Dataset | Downsampled to **1,200** records (SVR too slow on full data with GridSearch) |
| Parameters | `kernel` (RBF, Linear) · `C` (5 log-spaced values: 10¹–10⁴) · `epsilon` (0.1, 0.5) |
| Total fits | **100** |
| Best parameters | C = **10,000** · epsilon = **0.1** · kernel = **Linear** |
| Best CV R² | **0.21** |

**Test Results:** R² = **0.15** · RMSE = **$1,053,912** · MAE = **$498,295**

> SVR's weak performance reflects both the downsampling constraint and its known sensitivity to large-scale regression targets with outlier-heavy distributions.

---

#### 4. Random Forest Regressor ⭐ Best Regression Model

| Setting | Detail |
|---------|--------|
| CV | GridSearchCV · 3 folds |
| Parameters | `max_depth` (10, 20) · `n_estimators` (100, 200) |
| Best parameters | max_depth = **10** · n_estimators = **200** |
| Best CV R² | **0.58** |

**Test Results:** R² = **0.58** · RMSE = **$664,497** · MAE = **$338,598** ← Best overall

---

#### 5. Gradient Boosting Regressor

| Setting | Detail |
|---------|--------|
| Parameters | depth = 4 · max estimators = 500 · early stopping patience = 10 |
| Trees used | **75** (early stopping) |
| Train R² | 0.66 |

**Test Results:** R² = **0.57** · RMSE = **$673,340** · MAE = **$352,267**

---

#### 6. Neural Network

| Setting | Detail |
|---------|--------|
| Architectures tested | 3 |
| Best architecture | **4 hidden layers** — 256 → 128 → 64 → 32 → 1 |
| Target scaling | Sale Price ÷ 1,000,000 (scaled to million-dollar units for stable training) |
| Batch size | 512 |
| Callbacks | EarlyStopping(patience=15, restore_best_weights=True) |
| Epochs used | **84** (early stopping) |

- Target scaled to million-dollar units during training; predictions multiplied back after inference
- Validation loss tracked closely with training loss — less overfitting than the classification NN

**Test Results:** R² = **0.55** · RMSE = **$686,604** · MAE = **$342,118**

---

### 📈 Property Price Prediction — Full Model Comparison

| Model | CV R² | Test R² | RMSE | MAE |
|-------|-------|---------|------|-----|
| Linear Regression (Ridge) | 0.40 | 0.36 | $818,722 | $412,614 |
| Decision Tree | 0.52 | 0.51 | $718,495 | $371,546 |
| SVR | 0.21 | 0.15 | $1,053,912 | $498,295 |
| **Random Forest** | **0.58** | **0.58** | **$664,497** | **$338,598** |
| Gradient Boosting | — | 0.57 | $673,340 | $352,267 |
| Neural Network | — | 0.55 | $686,604 | $342,118 |

---

### 🏆 Property Price Prediction — Conclusion

**Random Forest** is the best regression model — achieving the highest test R² of **0.58** with RMSE of **$664,497** and MAE of **$338,598**.

Key findings:
- **Ensemble methods** (Random Forest, Gradient Boosting, Neural Network) clustered together in the R² 0.55–0.58 range, outperforming simpler models by a wide margin
- **SVR's poor performance** (R² = 0.15) is attributable to the mandatory 1,200-record downsample — a computational constraint that severely limited its ability to learn from the full feature space
- **Linear Regression** (R² = 0.36) reveals that price is not linearly separable from the available features — location encoded as borough is categorical and loses important intra-borough variation
- **Neural Network** (R² = 0.55) is competitive with Gradient Boosting but required target rescaling and careful training setup to converge on tabular data
- **RMSE of ~$664K** on a median sale price of $610K reflects the inherent heterogeneity of NYC real estate — properties in the same borough and tax class can differ by millions based on factors not captured in this dataset (floor, view, condition, proximity to subway)

**Future work:** Incorporating granular location features (neighborhood, block, floor level), property condition indicators, and proximity to transit may substantially improve price prediction accuracy.

---

---

## 🛠️ Tools & Libraries

| Category | Tools |
|----------|-------|
| **Language** | Python 3 |
| **Data** | Pandas · NumPy |
| **ML** | Scikit-learn (Ridge, LogisticRegression, DecisionTree, SVC/SVR, RandomForest, GradientBoosting, GridSearchCV) |
| **Imbalance** | Imbalanced-learn (SMOTE · down-sampling pipeline) |
| **Deep Learning** | TensorFlow / Keras (early stopping · target scaling for regression) |
| **Preprocessing** | StandardScaler · OneHotEncoder · Log Transformation · Pipeline · ColumnTransformer |
| **Visualization** | Matplotlib · Seaborn |
| **Environment** | Jupyter Notebook |

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd NYC-Borough-Prediction
pip install -r requirements.txt
jupyter notebook
```

**Required packages:**
```
pandas numpy scikit-learn imbalanced-learn tensorflow matplotlib seaborn
```

The notebook runs both sections end-to-end — the price prediction section reuses the same cleaned `df` and preprocessing pipeline from the borough section, so no separate data loading step is needed.

---

## 📁 File Structure

```
NYC-Borough-Prediction/
├── NYC_Borough_Prediction.ipynb    # Main notebook (Part 1: Borough + Part 2: Price)
├── NYC_Property_Sales.csv          # Dataset
└── README.md                       # This file
```

---

## 🔑 Key Takeaways

**Borough Classification:**
- **Sale Price and Land Square Feet** are the strongest predictors of borough — reflecting NYC's dramatically different real estate prices across boroughs
- **Manhattan at only 2% of data** is the hardest class to predict — severe underrepresentation leads to poor recall for this borough
- **Balancing did not help Logistic Regression** — the unbalanced model outperformed the SMOTE-balanced one

**Property Price Prediction:**
- **$10K–$10M price filter** is critical — leaving in $0 administrative transfers or $500M institutional sales would collapse model performance
- **BOROUGH as a feature** carries significant predictive signal: Manhattan properties command dramatically higher prices than the outer boroughs
- **Random Forest and Gradient Boosting** are robust to the skewed, heterogeneous nature of real estate prices — their non-parametric, tree-based structure handles outliers better than linear or kernel methods
- **SVR requires full-data training** to be competitive; the 1,200-record constraint makes its results non-representative of its true capability
- **Neural Network required target rescaling** — training directly on raw sale prices (in hundreds of thousands to millions) causes unstable gradients; dividing by 1,000,000 before training and rescaling predictions back resolved this

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
