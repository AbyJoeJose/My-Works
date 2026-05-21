# 🛒 Walmart 2024–25 Sales Dashboard (R Shiny)

> An interactive Shiny dashboard visualizing **Walmart customer purchase amounts by product category** — with 5 toggleable demographic and transaction filters (Age Group, City, Gender, Repeat Customer, Discount Applied) that dynamically update a bar chart in real time using reactive programming.

**Authors:** Aby Joe Jose & Meenakshy Manju · **Course:** IA 640 · **Date:** April 18, 2025

---

## 📌 Problem Statement

How do Walmart product category sales vary across different customer segments? This dashboard enables analysts to explore total purchase amounts by category — filtered interactively by customer demographics and transaction attributes — to identify which categories drive revenue for specific groups.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| File | `Walmart_customer_purchases.csv` |
| Source | Kaggle — Walmart Customer Purchases |
| Period | **2024–2025** |
| Categorical variables | `Customer_ID`, `Gender`, `City`, `Category`, `Product_Name`, `Purchase_Date`, `Payment_Method`, `Discount_Applied`, `Repeat_Customer` |
| Quantitative variables | `Age`, `Purchase_Amount`, `Rating` |

---

## 🔧 Data Preparation

### Step 1 — Remove single-category cities

Cities that sell in only one category are dropped — they add no comparative value for cross-category analysis:

```r
data <- data %>%
  group_by(City) %>%
  filter(n_distinct(Category) > 1) %>%
  ungroup()
```

### Step 2 — Drop `Customer_ID`

Removed because it holds unique values per row and contributes no meaningful insight for group-level analysis:

```r
data <- data %>% select(-Customer_ID)
```

### Step 3 — Create `AgeGroup`

The continuous `Age` column is binned into 6 categorical ranges using `cut()`:

```r
data$AgeGroup <- cut(data$Age,
                     breaks = c(0, 20, 30, 40, 50, 60, Inf),
                     labels = c('0-20', '20-30', '30-40', '40-50', '50-60', '66+'),
                     right = TRUE)
```

---

## 🖥️ Shiny Application

### UI — `fluidPage` with Sidebar Layout

The sidebar contains **5 filters**, each paired with a `checkboxInput("lock_*")` toggle that enables or disables it:

| Filter | Widget | Input type |
|--------|--------|-----------|
| Age Group | `selectInput` (multi) | `levels(data$AgeGroup)` |
| City | `selectizeInput` (multi) | `unique(data$City)` |
| Gender | `selectInput` (multi) | `unique(data$Gender)` |
| Repeat Customer | `selectInput` (single) | `unique(data$Repeat_Customer)` |
| Discount Applied | `selectInput` (single) | `unique(data$Discount_Applied)` |

`shinyjs::useShinyjs()` is declared in the UI to enable the `disable()` / `enable()` server-side calls.

**Note:** `selectizeInput` is used for City (vs `selectInput` for others) — `selectizeInput` supports better UX for large choice lists with search/autocomplete.

---

### Server — Reactive Logic

**Filter toggling** — 5 `observe()` blocks, one per filter, watch each `lock_*` checkbox and call `shinyjs::disable()` or `shinyjs::enable()` on the corresponding input:

```r
observe({
  if (input$lock_age) disable("selected_age")
  else enable("selected_age")
})
```

**Reactive data pipeline** — `filtered_data()` is a single reactive expression that applies all 5 filters conditionally. A filter is only applied when its checkbox is **not** locked (`!input$lock_*`):

```r
filtered_data <- reactive({
  filtered <- data

  if (!input$lock_city)     filtered <- filtered %>% filter(City %in% input$selected_city)
  if (!input$lock_age)      filtered <- filtered %>% filter(AgeGroup %in% input$selected_age)
  if (!input$lock_gender)   filtered <- filtered %>% filter(Gender %in% input$selected_gender)
  if (!input$lock_repeated) filtered <- filtered %>% filter(Repeat_Customer %in% input$selected_repeated)
  if (!input$lock_discount) filtered <- filtered %>% filter(Discount_Applied %in% input$selected_discount)

  filtered %>%
    group_by(Category) %>%
    summarise(Purchase_Amount = sum(Purchase_Amount, na.rm = TRUE), .groups = "drop")
})
```

The reactive chain ends by grouping by `Category` and summing `Purchase_Amount` — this is what feeds the bar chart.

---

### Output — Dynamic Bar Chart

A `renderPlot()` block renders the bar chart from `filtered_data()`. It updates automatically whenever any filter changes:

```r
ggplot(filtered_data(),
       aes(x = factor(Category), y = Purchase_Amount, fill = Category)) +
  geom_bar(stat = 'identity', width = 0.2) +
  scale_y_continuous(labels = label_number(scale_cut = cut_short_scale())) +
  theme_minimal() + ...
```

**Design choices:**

| Choice | Reason |
|--------|--------|
| `geom_bar(stat = 'identity')` | Data is pre-aggregated in `filtered_data()` — identity maps values directly without re-counting |
| `width = 0.2` | Narrow bars give visual breathing room between categories |
| `scale_y_continuous(label_number(scale_cut = cut_short_scale()))` | Y-axis auto-formats large values (e.g. 500,000 → 500K, 1,200,000 → 1.2M) |
| `fill = Category` | Each category gets a distinct color, reinforcing the x-axis grouping visually |
| `theme_minimal()` | Clean grid-focused theme; all axis text sized explicitly (title: 18pt, axis titles: 15pt, axis text: 12pt) |
| Dynamic `subtitle` | When all filters are locked → "Showing all data"; otherwise builds a descriptive string listing every active filter and its selection |

**Dynamic subtitle logic** — a nested `ifelse` checks all 5 lock flags; if all locked shows "Showing all data", otherwise assembles an active-filter summary:

```r
subtitle = ifelse(
  input$lock_age & input$lock_city & input$lock_gender & input$lock_repeated & input$lock_discount,
  "Showing all data",
  paste("Filtered by",
        ifelse(!input$lock_age,      paste("Age Group:", paste(input$selected_age, collapse = ",")), ""),
        ifelse(!input$lock_city,     paste("City:", paste(input$selected_city, collapse = ",")), ""),
        ifelse(!input$lock_gender,   paste("Gender:", paste(input$selected_gender, collapse = ",")), ""),
        ifelse(!input$lock_repeated, paste("Repeated Customer:", input$selected_repeated), ""),
        ifelse(!input$lock_discount, paste("Discount Applied:", input$selected_discount), "")
  )
)
```

---

## ⚠️ Key Challenge

The main engineering challenge was managing the **interplay between multiple filters** — ensuring that:

- Disabled filters are fully ignored in the data pipeline (not applied as empty selections)
- The reactive chain updates correctly when any single filter is toggled or its selection changes
- `shinyjs::disable()` visually grays out the input widget, while the server logic independently skips the filter using the `!input$lock_*` condition

A naive implementation would apply an empty filter (matching zero rows) when a user disables a widget without first selecting values. The `lock_*` flag cleanly separates UI state (disabled/enabled) from data filtering logic.

---

## 🛠️ Tools & Libraries

| Package | Purpose |
|---------|---------|
| `shiny` | Reactive web application framework — `fluidPage`, `sidebarLayout`, `selectInput`, `selectizeInput`, `checkboxInput`, `renderPlot`, `reactive`, `observe` |
| `shinyjs` | JavaScript-powered UI control — `useShinyjs()`, `disable()`, `enable()` for toggling filter inputs |
| `ggplot2` | Bar chart — `geom_bar(stat='identity')`, `facet_grid`, `theme_minimal`, `labs` |
| `scales` | Y-axis formatting — `label_number(scale_cut = cut_short_scale())` for K/M abbreviations |
| `dplyr` | Data wrangling — `group_by`, `filter`, `n_distinct`, `summarise`, `select`, `ungroup` |
| `readr` | CSV import |

---

## 🚀 How to Run

```r
# Install required packages if needed
install.packages(c("shiny", "shinyjs", "ggplot2", "dplyr", "scales", "readr"))

# Place 'Walmart_customer_purchases.csv' in your working directory
# Open Project_2.Rmd in RStudio and run the shinyApp() chunk
# Or run directly:
shiny::runApp("app.R")
```

---

## 📁 File Structure

```
Walmart-Sales-RShiny/
├── Project_2.Rmd                       # Main R Notebook with Shiny app
├── Walmart_customer_purchases.csv      # Dataset
└── README.md                           # This file
```

---

## 🔑 Key Takeaways

- **`lock_*` checkbox pattern** — pairing each filter with a disable toggle is more user-friendly than requiring users to clear selections; it preserves their last selection while temporarily showing all data for that dimension
- **`n_distinct(Category) > 1` filter** — removing single-category cities before the app loads prevents misleading comparisons where a city appears to "only sell" one category due to data sparsity rather than actual purchasing behavior
- **`selectizeInput` for City** — using `selectizeInput` instead of `selectInput` for the city filter gives users a searchable dropdown, essential when the number of unique cities is large
- **Pre-aggregation inside `reactive()`** — grouping and summing `Purchase_Amount` by `Category` inside the reactive expression (not inside `renderPlot`) keeps the plot code clean and separates data logic from presentation logic
- **`scale_cut = cut_short_scale()`** — automatically abbreviates axis labels (K, M, B) regardless of the magnitude of filtered data, so the chart stays readable whether filtered to a small city segment or the full dataset
- **Dynamic subtitle as a filter audit trail** — the subtitle tells the user exactly which filters are active and what values are selected, eliminating confusion about why the chart looks the way it does

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
