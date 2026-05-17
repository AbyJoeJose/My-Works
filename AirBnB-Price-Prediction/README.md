# 🏠 AirBnB Price Prediction & Classification

> An end-to-end machine learning project on 50,000+ New York City Airbnb listings — first predicting the exact nightly price using regression models, then classifying listings as Affordable (≤$500) or Expensive (>$500) using binary classifiers.

---

## 📌 Problem Statement

With over 50,000 active listings in New York City, pricing an Airbnb correctly is critical for hosts. Overpricing leads to empty calendars; underpricing leaves money on the table.

This project tackles the problem in two stages:

- **Part 1 — Regression:** Predict the exact nightly price as a continuous value
- **Part 2 — Classification:** Classify listings into Affordable (≤$500) vs Expensive (>$500)

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | Inside Airbnb (NYC open dataset) |
| Size | 50,000+ listings |
| Regression Target | `price` (nightly rate in USD) |
| Classification Target | `price_range` (Affordable / Expensive) |
| Features | Borough, neighbourhood, room type, host stats, reviews, availability |

---

## 🧠 Approach

### Data Preparation
1. **EDA** — Price distributions, correlation heatmap, outlier analysis (capped at $1,000)
2. **Preprocessing** — Missing value imputation, duplicate removal, datetime conversion
3. **Encoding** — One-hot encoding for `neighbourhood_group` and `room_type`
4. **Normality Testing** — Shapiro-Wilk test on numeric predictors
5. **Statistical Testing** — Kruskal-Wallis test to identify significant features

---

### Part 1: Price Prediction (Regression)

| Step | Detail |
|------|--------|
| Feature Scaling | StandardScaler |
| Train/Test Split | 80/20 stratified |
| Models | Linear Regression · Random Forest · XGBoost |
| Tuning | GridSearchCV (5-fold CV) on Random Forest and XGBoost |
| Evaluation | RMSE · MAE · R² |
| Visuals | Actual vs Predicted · Residuals vs Predicted · Residual Distribution |

**Mathematical foundations covered:**
- Linear Regression: Normal Equation $\hat{\boldsymbol{\beta}} = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{y}$
- Random Forest: Bootstrap aggregation $\hat{y}_{RF} = \frac{1}{B}\sum_{b=1}^{B} T_b(\mathbf{x})$
- XGBoost: Gradient boosting with regularisation $\mathcal{L}^{(t)} = \sum l(y_i, \hat{y}_i^{(t-1)} + f_t(x_i)) + \Omega(f_t)$

---

### Part 2: Price Range Classification

| Step | Detail |
|------|--------|
| Class Imbalance | SMOTE (oversampling minority class to 10,000) |
| Dimensionality Reduction | PCA (Scree plot, explained variance analysis) |
| Train/Test Split | 80/20 stratified on PCA components |
| Models | Logistic Regression · Decision Tree · Random Forest |
| Tuning | GridSearchCV (5-fold CV) on all three models |
| Evaluation | Accuracy · Precision · Recall · F1 · ROC-AUC |
| Visuals | Confusion Matrix · ROC Curves · Feature Importance · Decision Tree diagram |
---

## 📈 Results

### Regression

| Model | RMSE | MAE | R² |
|-------|------|-----|----|
| Linear Regression | baseline | baseline | baseline |
| Random Forest (Tuned) | ↓ improved | ↓ improved | ↑ improved |
| XGBoost (Tuned) | best | best | best |

### Classification

| Model | Accuracy | F1 | ROC-AUC |
|-------|----------|----|---------|
| Logistic Regression | baseline | — | — |
| Decision Tree (GridSearch) | — | — | — |
| Random Forest (GridSearch) | best | best | best |

---

## 🛠️ Tools & Libraries

| Category | Tools |
|----------|-------|
| **Language** | Python 3 |
| **Data** | Pandas · NumPy |
| **ML** | Scikit-learn · XGBoost · imbalanced-learn (SMOTE) |
| **Visualization** | Matplotlib · Seaborn |
| **Statistics** | SciPy (Shapiro-Wilk, Kruskal-Wallis) |
| **Environment** | Jupyter Notebook |

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd AirBnB-Price-Prediction
pip install -r requirements.txt
jupyter notebook AirBnB_Price_Analysis_Complete.ipynb
```

---

## 📁 File Structure

```
AirBnB-Price-Prediction/
├── AirBnB_Price_Analysis_Complete.ipynb   # Full notebook (regression + classification)
├── airbnb.csv                             # Dataset
└── README.md                             # This file
```

---

## 🔑 Key Takeaways

- Manhattan listings command the highest prices; Staten Island the lowest
- Room type is the strongest predictor in both regression and classification
- XGBoost outperforms Linear Regression and Random Forest on RMSE and R²
- SMOTE effectively addresses the severe class imbalance (Affordable >> Expensive)
- PCA reduces dimensionality while retaining the majority of explained variance
- Random Forest classifier achieves the best ROC-AUC among classification models

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
