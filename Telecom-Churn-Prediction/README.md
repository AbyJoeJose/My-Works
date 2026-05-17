# 📱 Telecom Customer Churn Prediction

> Identifying customers likely to cancel their telecom subscription, enabling proactive retention strategies.

---

## 📌 Problem Statement

Customer churn costs the telecom industry billions annually. Acquiring a new customer is 5–10x more expensive than retaining an existing one. This project builds a classifier to identify at-risk customers before they churn, giving the business time to intervene with targeted offers.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | IBM Telco Customer Churn Dataset |
| Size | ~7,000 customers |
| Target | `Churn` (Yes / No) |
| Features | Contract type, tenure, monthly charges, services subscribed, payment method |

---

## 🧠 Approach

1. **EDA** — Churn rate by contract type, tenure, and services
2. **Preprocessing** — Encoding, scaling, handling class imbalance (SMOTE)
3. **Modeling** — Logistic Regression, Random Forest, XGBoost, SVM
4. **Evaluation** — F1, ROC-AUC, Precision-Recall curve (priority: minimizing false negatives)

---

## 📈 Results

| Model | Accuracy | ROC-AUC |
|-------|----------|---------|
| Logistic Regression | baseline | — |
| Random Forest | — | best AUC |
| XGBoost | best F1 | — |

---

## 🛠️ Tools & Libraries

**Language:** Python · **Libraries:** Pandas, Scikit-learn, XGBoost, imbalanced-learn, Matplotlib, Seaborn

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd "Telecom-Churn-Prediction"
jupyter notebook
```

---

## 🔑 Key Takeaways

- Month-to-month contract customers churn at 3x the rate of annual contract customers
- High monthly charges combined with short tenure is the strongest churn signal
- Recall is prioritized over precision — missing a churner is more costly than a false alarm

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
