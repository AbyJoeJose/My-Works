# 🚲 Bike Sharing Demand Prediction

> Predicting daily bike rental counts using iterative OLS Linear Regression with RFE feature selection and VIF-based multicollinearity elimination — applied to 730 days of rental data across 2018–2019.

---

## 📌 Problem Statement

A bike-sharing company needs to understand which factors drive daily rental demand so they can optimise fleet allocation and maximise revenue. This project builds an interpretable linear regression model to predict the total daily bike count (`cnt`) from weather conditions, seasonality, and calendar features.

**Business value:** Accurate demand forecasting allows the company to pre-position bikes efficiently — reducing shortfalls on high-demand days and avoiding oversupply on low-demand ones.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| File | `day.csv` |
| Size | 730 daily records × 16 features |
| Period | 2018–2019 (2 years) |
| Target | `cnt` — total daily bike rentals (casual + registered) |
| Target Range | 22 – 8,714 rentals/day · Mean: 4,508 |

---

## 🧠 Approach

### 1. Data Cleaning & Feature Removal
- Dropped `instant` (row index — no predictive value) and `dteday` (date — superseded by `yr` and `mnth`)
- Dropped `casual` and `registered` — they are components of `cnt` (sum), so including them would be direct data leakage
- Dropped `temp` — extremely high correlation with `atemp` (felt temperature); `atemp` retained as the better predictor

### 2. Exploratory Data Analysis
- **Pairplot** — confirmed strong linear relationship between `cnt` and `atemp`, `casual`, `registered`
- **Correlation heatmap** — identified `temp`/`atemp` multicollinearity; `cnt` most correlated with `yr`, `atemp`, `season`
- **Box plots** — bike usage higher on working days; clear weather drives maximum demand; Light Snow/Rain dramatically reduces it
- **Month counts** — balanced distribution across months (56–62 records per month)

### 3. Feature Engineering — Dummy Variables
All categorical variables decoded and one-hot encoded with `drop_first=True`:

| Column | Mapping |
|--------|---------|
| `season` | 1→Spring, 2→Summer, 3→Fall, 4→Winter |
| `weekday` | 0→Sunday … 6→Saturday |
| `weathersit` | 1→Clear, 2→Misty/Cloudy, 3→Light Snow/Rain, 4→Heavy Rain |
| `mnth` | 1→January … 12→December |

Final dataset: **730 rows × 29 columns** before modeling

### 4. Train / Test Split
- **70/30 split** — 510 train · 220 test (`random_state=100`)
- MinMaxScaler applied to continuous features: `atemp`, `hum`, `windspeed`, `cnt`
- Scaler fitted on train set only; applied to test set to prevent data leakage

### 5. Feature Selection — RFE
- `LinearRegression` + `RFE(n_features_to_select=16)` on training data
- RFE selected: `yr`, `holiday`, `workingday`, `atemp`, `hum`, `windspeed`, `Summer`, `Winter`, `Light Snow/Rain`, `Misty/Cloudy weather`, `August`, `December`, `February`, `January`, `November`, `September`

### 6. Iterative OLS Modelling (LM1–LM10)
Ten OLS models built iteratively using `statsmodels`, removing features based on high VIF (multicollinearity) and high p-values at each step:

| Model | Action | R² | Adj. R² |
|-------|--------|-----|---------|
| LM1 | RFE top 16 features | 0.843 | 0.838 |
| LM2 | Remove `hum` (VIF=26.23) | 0.837 | 0.832 |
| LM3 | Remove `atemp` (VIF=5.90 after hum removed) | 0.771 | 0.764 |
| LM4 | Remove `workingday` (high p-value) | 0.769 | 0.763 |
| LM5 | Remove `windspeed` | 0.739 | 0.733 |
| LM6 | Add `Sunday` | 0.740 | 0.733 |
| LM7 | Remove `Winter` | 0.725 | 0.719 |
| LM8 | Remove `Summer` (high p-value) | 0.725 | 0.719 |
| LM9 | Add `Spring` | 0.778 | 0.772 |
| **LM10** | **Remove `Sunday` (high p-value) → Final model** | **0.777** | **0.772** |

---

## 🤖 Final Model — LM10

**11 predictors retained:**

| Feature | Coefficient | Interpretation |
|---------|-------------|---------------|
| `yr` | +0.2443 | Year-on-year demand growth |
| `Spring` | −0.1908 | Spring has lower demand vs Fall (base) |
| `Light Snow/Rain` | −0.3223 | Strongest negative driver — bad weather sharply reduces demand |
| `January` | −0.1441 | Coldest month — lowest demand |
| `February` | −0.0938 | Winter month — low demand |
| `November` | −0.0872 | Autumn decline |
| `December` | −0.0998 | Winter — reduced demand |
| `holiday` | −0.0818 | Holidays slightly reduce rentals |
| `Misty/Cloudy weather` | −0.0885 | Moderate weather reduces demand |
| `September` | +0.0984 | Late summer peak |
| `August` | +0.0586 | Summer peak month |

**All p-values < 0.05 · All VIF values < 3.2**

---

## 📈 Results

| Metric | Train | Test |
|--------|-------|------|
| R² | 0.777 | **0.780** |
| Adjusted R² | 0.772 | — |
| RMSE | — | **0.1025** (normalised) |
| MSE | — | 0.0105 (normalised) |

The test R² (0.780) is marginally **higher** than the train R² (0.777) — confirming the model generalises well and is not overfitting.

---

## 📊 Key Visualisations

- Pairplot of continuous variables (`atemp`, `hum`, `windspeed`, `casual`, `registered`, `cnt`)
- Correlation heatmap (before and after feature removal)
- Box plots: `cnt` vs `holiday`, `cnt` vs `weathersit`
- Residual distribution plot (error term normality check)
- Actual vs Predicted scatter plot (`y_test` vs `y_test_pred`)

---

## 🛠️ Tools & Libraries

| Category | Tools |
|----------|-------|
| **Language** | Python 3 |
| **Data** | Pandas · NumPy |
| **ML** | Scikit-learn (LinearRegression, RFE, MinMaxScaler) |
| **Statistics** | Statsmodels (OLS, VIF) |
| **Visualization** | Matplotlib · Seaborn |
| **Environment** | Jupyter Notebook |

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd Bike-Sharing-Profit-Prediction
pip install pandas numpy scikit-learn statsmodels matplotlib seaborn
jupyter notebook
```

Place `day.csv` in the same directory as the notebook before running.

---

## 📁 File Structure

```
Bike-Sharing-Profit-Prediction/
├── bike_sharing.ipynb      # Main notebook
├── day.csv                 # Dataset
└── README.md               # This file
```

---

## 🔑 Key Takeaways

- **Year (`yr`) is the strongest positive predictor** — demand grew significantly from 2018 to 2019, reflecting growing adoption of bike sharing
- **Light Snow/Rain has the largest negative impact** (coeff = −0.32) — bad weather is the single biggest deterrent to bike usage
- **Spring surprisingly shows lower demand than Fall** (the reference category) — Spring's coefficient is negative, suggesting Fall is the peak season
- **`temp` and `atemp` were highly correlated** — retaining both would have caused severe multicollinearity (VIF > 14); only `atemp` was kept in early models before further simplification
- **`hum` had the highest VIF (26.23)** — removed first due to its dominant multicollinearity with other weather features
- **Iterative VIF + p-value elimination** across 10 models produced a stable, interpretable final model with all VIF < 3.2 and all p-values < 0.05
- **Test R² (0.780) ≈ Train R² (0.777)** — the model generalises well with no signs of overfitting

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
