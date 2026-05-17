# 🚀 SpaceX Falcon 9 Launch Success Prediction

> Predicting whether a SpaceX Falcon 9 first-stage booster will successfully land and be recovered — enabling cost estimation for competitive rocket launches.

---

## 📌 Problem Statement

SpaceX advertises Falcon 9 launches at ~$62M, far cheaper than competitors, largely because of booster reusability. Predicting whether a first-stage booster will land successfully allows competing companies to estimate launch costs and bid more accurately. This project builds a binary classifier using real SpaceX launch data.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | SpaceX REST API + Wikipedia web scraping |
| Size | 90+ launches |
| Target | `landing_success` (1 = landed, 0 = failed) |
| Features | Payload mass, orbit type, launch site, booster version, flight number |

---

## 🧠 Approach

1. **Data Collection** — SpaceX REST API + BeautifulSoup web scraping
2. **EDA** — Launch site maps (Folium), success rates by orbit and booster version
3. **Feature Engineering** — One-hot encoding, payload bins, flight number normalization
4. **Modeling** — Logistic Regression, SVM, Decision Tree, KNN with GridSearchCV
5. **Evaluation** — Accuracy, Confusion Matrix, best hyperparameters

---

## 📈 Results

| Model | Test Accuracy |
|-------|--------------|
| Logistic Regression | ~83% |
| SVM | ~83% |
| Decision Tree | ~83% |
| KNN | ~83% |

*All models performed similarly — suggesting the dataset size limits further gains.*

---

## 🛠️ Tools & Libraries

**Language:** Python · **Libraries:** Pandas, NumPy, Scikit-learn, Folium, Plotly, Matplotlib, Requests, BeautifulSoup

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd "SpaceX-Launch-Success-Prediction"
jupyter notebook
```

---

## 🔑 Key Takeaways

- Launch site and orbit type are the strongest predictors of landing success
- Success rate has improved dramatically over time as SpaceX refined its technology
- Interactive Folium maps reveal geographic clustering of launch outcomes

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
