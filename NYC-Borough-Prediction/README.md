# 🗽 New York City Borough Prediction

> A multiclass classification project predicting which of New York City's five boroughs a property belongs to — using 6 models including Logistic Regression, Decision Tree, SVC, Random Forest, Gradient Boosting, and Neural Networks — applied to 84,548 NYC Property Sales transactions.

---

## 📌 Problem Statement

New York City is a rapidly growing urban center with significant opportunities for business and residential development. This project builds a multiclass classification model to predict the **borough** of a property based on features like tax class, sale price, building age, unit counts, and square footage.

**Target classes:** Brooklyn · Queens · Bronx · Staten Island · Manhattan

The model aims to support stakeholders in making informed and efficient decisions for business or residential planning in NYC's complex real estate market.

---

## 📊 Dataset

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

## 🧠 Approach

### 1. Exploratory Data Analysis
- All numerical features are **right-skewed** in distribution
- Residential units significantly outnumber commercial units relative to total units
- Borough target variable is **heavily imbalanced** — Brooklyn dominates at 48%

### 2. Data Preparation & Preprocessing

**Feature Engineering:**
- `Year Built` → converted to **building age** by subtracting from current year (2025)
- Removed address and zip code columns to prevent **horizontal data leakage**
- Removed redundant columns and high-cardinality categorical features
- No text or time features — no vectorizer needed

**Train / Validation / Test Split:**
- **70%** training · **15%** validation · **15%** test (equal split of remaining 30%)

**Numerical Pipeline:**
1. Median imputation for missing values
2. Log transformation to reduce right skewness
3. Standard Scaler for normalisation

**Categorical Pipeline:**
1. Most-frequent imputation for nulls
2. One-Hot Encoding

**Class Imbalance Handling:**
- Separate **Imblearn pipeline** for Logistic Regression: down-sampling first, then SMOTE oversampling
- After balancing: all 5 classes have **2,700 rows** each
- All other models trained on the **unbalanced** dataset

---

## 🤖 Models

### 1. Logistic Regression (Balanced & Unbalanced)

| Setting | Detail |
|---------|--------|
| CV | GridSearchCV · 5 folds |
| Parameter | `C` — 10 log-spaced values between 10⁻³ and 10² |
| Best C | **27.82** (same for both balanced and unbalanced) |

- Unbalanced dataset showed **lower standard deviation** in validation scores
- Unbalanced model **outperformed** the balanced model — a surprising finding

| Model | F1 (macro) | Accuracy |
|-------|-----------|---------|
| Logistic Regression (Balanced) | 0.47 | 0.53 |
| Logistic Regression (Unbalanced) | **0.48** | **0.63** |

---

### 2. Decision Tree Classifier

| Setting | Detail |
|---------|--------|
| Dataset | Unbalanced |
| CV | GridSearchCV · 5 folds |
| Parameter | `max_depth` — values 1 to 15 |
| Best max_depth | **13** |
| Best train F1 | 0.67 |

- Model begins to overfit as max depth increases beyond 13

**Test Results:** Accuracy = **0.72** · F1 (macro) = **0.68**

---

### 3. Support Vector Classifier (SVC)

| Setting | Detail |
|---------|--------|
| Dataset | Resampled to **1,000** records (SVC too slow on full data with GridSearch) |
| Parameters tuned | `C` (10 log-spaced values: 0.1–3) · `Gamma` (10 values: 0.1–5) · `Kernel` (RBF, Poly) |
| Total fits | **1,000** |
| Best parameters | C = **51.41** · Gamma = **5.0** · Kernel = **RBF** |
| Best train F1 | 0.55 |

**Test Results:** Accuracy = **0.63** · F1 (macro) = **0.53**

---

### 4. Random Forest ⭐ Best Model

| Setting | Detail |
|---------|--------|
| Dataset | Unbalanced |
| CV | GridSearchCV |
| Parameters | `max_depth` (10–30) · `n_estimators` (100–1000) |
| Total fits | **225** |
| Best parameters | max_depth = **20** · n_estimators = **700** |
| Best train F1 | **0.75** |

**Top Features:** `Sale Price` and `Land Square Feet` contribute most to the model.

**Test Results:** Accuracy = **0.76** · F1 (macro) = **0.72** ← Best overall

---

### 5. Gradient Boosting

| Setting | Detail |
|---------|--------|
| Parameters | depth = 4 · estimators = 500 |
| Early stopping | 10 iterations with no change |
| Trees used | **14 trees** |
| Best train F1 | 0.71 |

**Test Results:** Accuracy = **0.73** · F1 (macro) = **0.67**

---

### 6. Neural Network

| Setting | Detail |
|---------|--------|
| Models tested | 5 different architectures |
| Callbacks | Patience = 10 (early stopping) |
| Best architecture | **5 hidden layers** — 100 neurons (initial layers) tapering to 10 neurons (later layers) |
| Best train loss | 0.65 |

- Validation loss decrease less evident than training loss — signs of mild overfitting

**Test Results:** Accuracy = **0.74** · F1 (macro) = **0.68**

---

## 📈 Full Model Comparison

| Model | Train F1 | Test F1 | Test Accuracy |
|-------|---------|---------|--------------|
| Logistic Regression (Unbalanced) | 0.48 | 0.48 | 0.63 |
| Decision Tree | 0.67 | 0.68 | 0.72 |
| SVC | 0.55 | 0.53 | 0.63 |
| **Random Forest** | **0.75** | **0.72** | **0.76** |
| Gradient Boosting | 0.71 | 0.67 | 0.73 |
| Neural Network | 0.65 | 0.68 | 0.72 |

---

## 🏆 Conclusion

**Random Forest** is the best overall model — achieving the highest test F1 macro score of **0.72** and test accuracy of **76%**. It provides the strongest balance between bias and variance for this multiclass task.

Key findings:
- Simpler models (Logistic Regression, SVC) struggled with the 5-class imbalanced problem
- Ensemble methods (Random Forest, Gradient Boosting) significantly outperformed linear and kernel-based models
- The balanced dataset did **not** improve Logistic Regression — the unbalanced version performed better, highlighting that balancing alone does not guarantee improvement for all model types
- Neural Networks showed strong potential but did not surpass Random Forest

**Future work:** Applying SMOTE to ensemble models may further improve performance, particularly for underrepresented boroughs like Manhattan (only 2% of data).

---

## 🛠️ Tools & Libraries

| Category | Tools |
|----------|-------|
| **Language** | Python 3 |
| **Data** | Pandas · NumPy |
| **ML** | Scikit-learn (LogisticRegression, DecisionTree, SVC, RandomForest, GradientBoosting, GridSearchCV) |
| **Imbalance** | Imbalanced-learn (SMOTE, down-sampling pipeline) |
| **Deep Learning** | TensorFlow / Keras (Neural Networks with early stopping callbacks) |
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

---

## 📁 File Structure

```
NYC-Borough-Prediction/
├── nyc_borough_prediction.ipynb    # Main notebook
├── Final_Report.pdf                # Project report
├── nyc_property_sales.csv          # Dataset
└── README.md                       # This file
```

---

## 🔑 Key Takeaways

- **Sale Price and Land Square Feet** are the strongest predictors of borough — reflecting NYC's dramatically different real estate prices across boroughs
- **Manhattan at only 2%** of the data is the hardest class to predict — severe underrepresentation leads to poor recall for this borough
- **Balancing did not help Logistic Regression** — the unbalanced model outperformed the SMOTE-balanced one, showing that more data does not always compensate for model complexity limitations
- **SVC required downsampling to 1,000 records** due to computational cost with GridSearchCV — a practical constraint that affected its final performance
- **Random Forest with depth=20 and 700 estimators** found the optimal bias-variance tradeoff among all models tested
- **Neural network validation loss** did not decrease as clearly as training loss — indicating that more regularisation or data augmentation would be beneficial for deep learning on this dataset

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
