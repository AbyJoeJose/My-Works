# 🌦️ Australia Weather Prediction — Melbourne Rainfall Classifier

> Predicting daily rainfall in the Melbourne area using a scikit-learn pipeline combining Random Forest and Logistic Regression classifiers, optimized with GridSearchCV and StratifiedKFold cross-validation on 7,557 weather observations from 2008–2017.

---

## 📌 Problem Statement

Given historical weather observations up to and including yesterday, can we predict whether it will rain **today** in the Melbourne area? This localized approach avoids data leakage from same-day measurements and has direct practical applications — for example, deciding whether to bike to work.

**Target variable:** `RainToday` (Yes / No)  
**Key insight:** A naive baseline that always predicts "No Rain" would be correct ~76% of the time — so a good model must meaningfully exceed this.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | Australian Bureau of Meteorology via Kaggle |
| Original Size | 145,460 observations × 23 features (2008–2017) |
| After Dropping Missing Values | 56,420 observations |
| After Location Filter | **7,557 observations** (Melbourne region only) |
| Target | `RainToday` (Yes = 23.7% · No = 76.3%) |
| Locations | Melbourne · MelbourneAirport · Watsonia |

**Why filter to Melbourne?**  
Weather patterns vary significantly across Australia. Grouping three geographically close locations (within 18km of each other) produces a more locally accurate model while retaining enough data to train reliably.

---

## 🧠 Approach

### 1. Data Leakage Handling
- Columns like `Rainfall` and `RainToday` describe the full day's observations — using them to predict today's rain would be leakage
- Reframed as: **predict today's rain using yesterday's data**
- Renamed `RainToday → RainYesterday` and `RainTomorrow → RainToday` to reflect this

### 2. Missing Value Strategy
- `Sunshine` and `Cloud` columns had very high missing rates across the full dataset
- Strategy: **drop all rows with any missing values** — 56,420 clean records remain (sufficient for modelling)
- No imputation required

### 3. Feature Engineering — Seasonality
- Extracted `Season` from the `Date` column using Australian seasons:
  - Summer: Dec, Jan, Feb
  - Autumn: Mar, Apr, May
  - Winter: Jun, Jul, Aug
  - Spring: Sep, Oct, Nov
- Dropped `Date` after extraction (less informative than season)

### 4. Class Imbalance
- Target is imbalanced: 76.3% No Rain vs 23.7% Rain
- Used **stratified train/test split** (`stratify=y`) to preserve class ratio
- Logistic Regression also tested with `class_weight='balanced'`

### 5. Preprocessing Pipeline
- **Numeric features** (16): StandardScaler
- **Categorical features** (6 — Location, WindGustDir, WindDir9am, WindDir3pm, RainYesterday, Season): OneHotEncoder (`handle_unknown='ignore'`)
- Combined using `ColumnTransformer` inside a `Pipeline`

### 6. Model Optimization
- `GridSearchCV` with `StratifiedKFold(n_splits=5, shuffle=True)`
- Ensures class balance is maintained across all CV folds

---

## 🤖 Models

### Model 1 — Random Forest Classifier

**Parameter Grid Searched:**

| Parameter | Values Tried |
|-----------|-------------|
| `n_estimators` | 50, 100 |
| `max_depth` | None, 10, 20 |
| `min_samples_split` | 2, 5 |

**Total CV fits:** 12 combinations × 5 folds = **60 fits**

**Best Parameters:**
```
max_depth=20 · min_samples_split=2 · n_estimators=100
```

**Best CV Score:** 0.85

**Test Set Results:**

| Metric | No Rain | Rain |
|--------|---------|------|
| Precision | 0.86 | 0.75 |
| Recall | 0.95 | 0.51 |
| F1-Score | 0.90 | 0.61 |

**Overall Test Accuracy: 84%**

**Confusion Matrix:**
```
              Predicted No   Predicted Yes
Actual No         1096            58
Actual Yes         174           184
```

**True Positive Rate (Rain correctly predicted): 51%**

---

### Model 2 — Logistic Regression

**Parameter Grid Searched:**

| Parameter | Values Tried |
|-----------|-------------|
| `solver` | liblinear |
| `penalty` | l1, l2 |
| `class_weight` | None, balanced |

**Total CV fits:** 4 combinations × 5 folds = **20 fits**

**Test Set Results:**

| Metric | No Rain | Rain |
|--------|---------|------|
| Precision | 0.86 | 0.69 |
| Recall | 0.93 | 0.51 |
| F1-Score | 0.89 | 0.59 |

**Overall Test Accuracy: 83%**

**True Positive Rate (Rain correctly predicted): 51%**

---

## 📈 Model Comparison

| Metric | Random Forest | Logistic Regression |
|--------|--------------|-------------------|
| Test Accuracy | **84%** | 83% |
| Rain Precision | **0.75** | 0.69 |
| Rain Recall | 0.51 | 0.51 |
| Rain F1 | **0.61** | 0.59 |
| No Rain F1 | **0.90** | 0.89 |
| Correct Predictions | 1,280 / 1,512 | 1,255 / 1,512 |

**Winner: Random Forest** — marginally better accuracy and precision on the minority class (Rain).  
Both models share the same true positive rate of 51% for rainfall prediction.

---

## 📊 Feature Importances (Random Forest)

Top predictive features identified from the best Random Forest model:

| Rank | Feature | Why It Matters |
|------|---------|---------------|
| 1 | **Humidity3pm** | Most important — afternoon humidity strongly signals incoming rain |
| 2 | Sunshine | Fewer sunshine hours correlates with rainy conditions |
| 3 | Pressure3pm | Falling pressure is a classic rain indicator |
| 4 | Cloud3pm | Higher afternoon cloud cover precedes rain |
| 5 | Humidity9am | Morning humidity provides early signal |

Feature importances were extracted from `best_estimator_['classifier'].feature_importances_` and matched back to original feature names through the one-hot encoder pipeline for categorical variables.

---

## 🛠️ Tools & Libraries

| Category | Tools |
|----------|-------|
| **Language** | Python 3 |
| **Data** | Pandas · NumPy |
| **ML Pipeline** | Scikit-learn — Pipeline · ColumnTransformer |
| **Models** | RandomForestClassifier · LogisticRegression |
| **Tuning** | GridSearchCV · StratifiedKFold |
| **Preprocessing** | StandardScaler · OneHotEncoder |
| **Evaluation** | classification_report · confusion_matrix · ConfusionMatrixDisplay |
| **Visualization** | Matplotlib · Seaborn |
| **Environment** | Jupyter Notebook |

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd Australia-Weather-Prediction
pip install -r requirements.txt
jupyter notebook
```

**Required packages:**
```
pandas numpy scikit-learn matplotlib seaborn
```

The dataset is loaded directly from a public URL — no manual download needed:
```python
url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/_0eYOqji3unP1tDNKWZMjg/weatherAUS-2.csv"
```

---

## 📁 File Structure

```
Australia-Weather-Prediction/
├── australia_weather.ipynb     # Main notebook
└── README.md                   # This file
```

---

## 🔑 Key Takeaways

- **Humidity at 3pm is the single strongest predictor** of rainfall — afternoon atmospheric moisture is the clearest signal available from prior-day data
- **Localizing to Melbourne** dramatically simplifies the modelling problem — using all of Australia requires a far more complex model to handle regional weather diversity
- **Season engineering** from the Date column captures cyclical weather patterns without leaking future information
- **A naive baseline** (always predict No Rain) achieves 76.3% accuracy — our best model at 84% represents a meaningful 8-point improvement
- **True positive rate of 51%** means the model catches about half of all rainy days — acceptable for general use but could be improved with more data or ensemble tuning
- **Both models agree** on true positive rate (51%) — suggesting this is a data limitation rather than a model limitation; more granular features or longer history may help

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
