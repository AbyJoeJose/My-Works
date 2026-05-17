# 🌦️ Australia Weather Prediction

> Binary classification model to predict whether it will rain tomorrow in Australia using historical weather observations.

---

## 📌 Problem Statement

Accurate rainfall prediction is vital for agriculture, water management, and emergency planning in Australia — one of the driest continents on Earth. This project builds a classifier to predict next-day rainfall from daily weather station data.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | Australian Bureau of Meteorology |
| Size | ~145,000 daily observations |
| Target | `RainTomorrow` (Yes / No) |
| Features | Temperature, humidity, wind speed, pressure, sunshine hours |

---

## 🧠 Approach

1. **EDA** — Seasonal patterns, missing value analysis, feature correlations
2. **Preprocessing** — Imputation, encoding, feature scaling
3. **Modeling** — Logistic Regression, Decision Tree, Random Forest, XGBoost
4. **Evaluation** — Accuracy, Precision, Recall, F1, ROC-AUC

---

## 📈 Results

| Model | Accuracy | ROC-AUC |
|-------|----------|---------|
| Logistic Regression | baseline | — |
| Random Forest | best F1 | — |
| XGBoost | best AUC | — |

---

## 🛠️ Tools & Libraries

**Language:** Python · **Libraries:** Pandas, Scikit-learn, XGBoost, Matplotlib, Seaborn

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd "Australia-Weather-Prediction"
jupyter notebook
```

---

## 🔑 Key Takeaways

- Humidity at 3pm is the single strongest predictor of next-day rain
- Class imbalance (~78% No Rain) requires careful evaluation metrics
- Ensemble models significantly outperform logistic regression baseline

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
