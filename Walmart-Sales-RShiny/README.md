# 🛒 Walmart Sales Dashboard (R Shiny)

> Interactive R Shiny web application to explore and visualize Walmart weekly sales trends across departments and stores, with dynamic filtering and forecasting.

---

## 📌 Problem Statement

Retail chains like Walmart need to understand sales dynamics across hundreds of stores and departments to optimize inventory, staffing, and promotions. Static reports fall short — this project builds an interactive dashboard that lets analysts explore the data dynamically.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | Kaggle — Walmart Store Sales Forecasting |
| Size | ~420,000 weekly sales records |
| Scope | 45 stores, 99 departments, 2.5 years |
| Features | Weekly sales, store, department, date, holiday flag, temperature, fuel price, CPI, unemployment |

---

## 🧠 Features of the Dashboard

- **Store & Department Filter** — Drill down into any store or department
- **Date Range Selector** — Zoom into any time window
- **Sales Trend Chart** — Weekly sales time series with holiday highlights
- **Department Comparison** — Bar chart of top-performing departments
- **Holiday Impact Analysis** — Compare holiday vs non-holiday sales
- **Summary Statistics** — KPI cards for total sales, average, and growth rate

---

## 🛠️ Tools & Libraries

**Language:** R · **Framework:** R Shiny · **Libraries:** ggplot2, dplyr, lubridate, plotly, shinydashboard

---

## 🚀 How to Run

```r
# Install required packages
install.packages(c("shiny", "ggplot2", "dplyr", "lubridate", "plotly", "shinydashboard"))

# Run the app
shiny::runApp("app.R")
```

---

## 🔑 Key Takeaways

- Holiday weeks (Thanksgiving, Christmas) produce 2–3x normal sales volume
- Department 72 (Electronics) shows the highest holiday lift
- Store size is a strong predictor of overall sales performance
- R Shiny enables non-technical stakeholders to explore data without code

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
