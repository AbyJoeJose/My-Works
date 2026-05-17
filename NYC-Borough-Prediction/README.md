# 🗽 NYC Borough Prediction

> Classifying New York City taxi pickups into the five boroughs using geospatial and time-based features.

---

## 📌 Problem Statement

New York City's taxi and ride-sharing ecosystem generates millions of trips daily. Predicting which borough a pickup belongs to — based on coordinates, time, and trip characteristics — is a geospatial classification problem with practical applications in routing and demand forecasting.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | NYC Taxi & Limousine Commission |
| Size | Millions of trip records |
| Target | Borough (Manhattan, Brooklyn, Queens, Bronx, Staten Island) |
| Features | Pickup coordinates, time of day, day of week, trip distance |

---

## 🧠 Approach

1. **EDA** — Pickup density maps, borough distribution, temporal patterns
2. **Feature Engineering** — Coordinate binning, time-of-day flags, distance features
3. **Modeling** — Logistic Regression, KNN, Random Forest, Gradient Boosting
4. **Evaluation** — Accuracy, F1 (macro), Confusion Matrix

---

## 📈 Results

| Model | Accuracy | F1 (Macro) |
|-------|----------|-----------|
| Logistic Regression | baseline | — |
| Random Forest | best performer | — |

---

## 🛠️ Tools & Libraries

**Language:** Python · **Libraries:** Pandas, Scikit-learn, Folium, Matplotlib, Seaborn

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd "NYC-Borough-Prediction"
jupyter notebook
```

---

## 🔑 Key Takeaways

- Latitude and longitude alone provide strong borough separation
- Manhattan dominates pickup volume; Staten Island is heavily underrepresented
- Class imbalance required stratified sampling for reliable evaluation

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
