# 🚲 Bike Sharing Profit Prediction

> Predicting hourly and daily bike rental demand to optimize fleet allocation and maximize revenue for a bike-sharing service.

---

## 📌 Problem Statement

Bike-sharing companies need to pre-position bikes efficiently across stations. Too few bikes at a busy station means lost revenue; too many at a quiet one wastes resources. This project forecasts rental demand using time, weather, and seasonal features.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | UCI Machine Learning Repository |
| Size | ~17,000 hourly records (2 years) |
| Target | `count` (total rentals per hour) |
| Features | Season, holiday, weather, temperature, humidity, wind speed |

---

## 🧠 Approach

1. **EDA** — Demand by hour, day, season; weather impact analysis
2. **Feature Engineering** — Time-based features (rush hour, weekend flags)
3. **Modeling** — Linear Regression, Random Forest, Gradient Boosting
4. **Evaluation** — RMSE, MAE, R²

---

## 📈 Results

| Model | RMSE | R² |
|-------|------|----|
| Linear Regression | baseline | — |
| Random Forest | best performer | — |
| Gradient Boosting | competitive | — |

---

## 🛠️ Tools & Libraries

**Language:** Python · **Libraries:** Pandas, NumPy, Scikit-learn, Matplotlib, Seaborn

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd "Bike-Sharing-Profit-Prediction"
jupyter notebook
```

---

## 🔑 Key Takeaways

- Demand peaks at 8am and 5-6pm on weekdays (commute patterns)
- Temperature is the strongest weather predictor of demand
- Tree-based models capture nonlinear time patterns that linear models miss

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
