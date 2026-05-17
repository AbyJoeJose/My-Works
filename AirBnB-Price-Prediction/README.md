# 🏠 AirBnB Price Prediction

> Predicting New York City Airbnb listing prices using machine learning regression models based on location, host attributes, and listing features.

---

## 📌 Problem Statement

With over 50,000 active listings in New York City, pricing an Airbnb correctly is critical for hosts. Overpricing leads to empty calendars; underpricing leaves money on the table. This project builds a regression model to predict the optimal nightly price for a listing based on its features.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | Inside Airbnb (NYC open dataset) |
| Size | 50,000+ listings |
| Target | `price` (nightly rate in USD) |
| Features | Location, room type, amenities, host stats, reviews |

---

## 🧠 Approach

1. **EDA** — Distribution of prices, geospatial analysis by borough, correlation heatmaps
2. **Preprocessing** — Outlier removal, encoding categorical variables, handling missing values
3. **Feature Engineering** — Distance to city center, host response rate binning
4. **Modeling** — Linear Regression, Random Forest, Gradient Boosting
5. **Evaluation** — RMSE and R² on held-out test set

---

## 📈 Results

| Model | RMSE | R² |
|-------|------|----|
| Linear Regression | baseline | — |
| Random Forest | best performer | — |
| Gradient Boosting | competitive | — |

---

## 🛠️ Tools & Libraries

**Language:** Python · **Libraries:** Pandas, NumPy, Scikit-learn, Matplotlib, Seaborn, Folium

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd "AirBnB-Price-Prediction"
jupyter notebook
```

---

## 🔑 Key Takeaways

- Manhattan listings command the highest prices; Staten Island the lowest
- Room type is the strongest predictor of price
- Geospatial features significantly improve model performance

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
