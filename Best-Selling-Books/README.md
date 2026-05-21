# 📚 Best Selling Books by 5 Leading Authors (R / Plotly)

> An interactive data visualization project in R analyzing Amazon bestselling books to identify the **top 5 authors** by a composite score and rank their **top 5 books each** — rendered as a faceted, interactive Plotly bar chart.

**Authors:** Aby Joe Jose & Meenakshy Manju · **Date:** April 18, 2025

---

## 📌 Problem Statement

Which authors dominate the Amazon bestseller list, and which of their books perform best? This project builds a composite **Score** metric from user ratings and review counts to rank authors and their books — then visualizes the results interactively so readers can explore price, genre, and exact scores on hover.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| File | `bestsellers with categories.csv` |
| Source | Amazon Bestselling Books dataset |
| Key columns | `Name`, `Author`, `User.Rating`, `Reviews`, `Genre`, `Price`, `Year` |
| Null values | **0** (confirmed with `sum(is.na(data1))`) |
| Raw rows | checked with `nrow(data1)` |

---

## 🧠 Methodology

### Step 1 — Deduplication

The raw dataset contains repeated entries for the same book across multiple years (same Name, User.Rating, Reviews, and Genre but different Year and Price). Duplicates are removed keeping only the **first occurrence** (which retains the first year and price seen):

```r
data_unique <- data1 %>%
  distinct(Name, User.Rating, Reviews, Genre, .keep_all = TRUE)
```

### Step 2 — Composite Score

A single **Score** column is created by combining `User.Rating` and `Reviews`:

```
Score = (User.Rating × 1,000) + Reviews
```

This weights the rating heavily (scaled to thousands) while using raw review count as a tiebreaker — books with higher ratings dominate, and among equally rated books, review volume decides rank.

### Step 3 — Top 5 Authors

Authors are ranked by **total Score** across all their unique books:

```r
Top_5_author <- data_unique %>%
  group_by(Author) %>%
  summarise(Total_Score = sum(Score)) %>%
  arrange(desc(Total_Score)) %>%
  head(5)
```

### Step 4 — Top 5 Books per Author

Within the filtered author subset, the top 5 books per author are selected by Score:

```r
Top_5_books_of_each_author <- data %>%
  group_by(Author) %>%
  arrange(desc(Score)) %>%
  slice_head(n = 5) %>%
  ungroup()
```

### Step 5 — Name Truncation

Book titles are trimmed to **25 characters** for clean axis labels:

```r
Top_5_books_of_each_author$ShortName <- substr(Top_5_books_of_each_author$Name, 1, 25)
```

---

## 📈 Visualization

An interactive **faceted horizontal bar chart** built with `ggplot2` and converted to Plotly:

```r
p <- ggplot(Top_5_books_of_each_author,
       aes(x = reorder(ShortName, Score),
           y = Score,
           fill = Author,
           text = paste("Name of the book:", Name,
                        '<br>Author:', Author,
                        '<br>Price: $', Price,
                        '<br>Genre:', Genre,
                        '<br>Score:', Score))) +
  geom_col() +
  scale_fill_brewer(type = "qual", palette = 'Set1') +
  facet_grid(Author ~ ., scales = "free_y") +
  theme_minimal() +
  labs(title = "Best Selling Books by 5 Leading Authors", x = NULL, y = 'Score') +
  coord_flip() +
  scale_y_continuous(labels = function(x) paste0((x / 1000), "k")) +
  theme(strip.text.y = element_blank(), strip.background = element_blank())

ggplotly(p, tooltip = 'text')
```

**Design choices:**

| Choice | Reason |
|--------|--------|
| `facet_grid(Author ~ ., scales = "free_y")` | Each author gets its own panel with an independently scaled y-axis so books aren't crowded |
| `strip.text.y = element_blank()` | Author name labels on strip removed — fill color + legend already identifies author |
| `reorder(ShortName, Score)` | Bars within each facet sorted by Score descending (highest at top) |
| `scale_y_continuous` with `paste0((x/1000), "k")` | Score axis displays in thousands (e.g. 5250 → 5.25k) for readability |
| `scale_fill_brewer(palette = 'Set1')` | Qualitative ColorBrewer palette — 5 distinct, colorblind-friendly colors |
| `coord_flip()` | Horizontal bars so long book names don't overlap |
| `ggplotly(tooltip = 'text')` | Custom hover tooltip shows full book name, author, price, genre, and score |

---

## 🛠️ Tools & Libraries

| Package | Purpose |
|---------|---------|
| `dplyr` | Data manipulation — `distinct()`, `group_by()`, `summarise()`, `arrange()`, `slice_head()` |
| `ggplot2` | Base chart — faceted bar plot with `geom_col()`, `facet_grid()`, `coord_flip()` |
| `plotly` | Interactive layer — `ggplotly()` with custom tooltip text |
| `RColorBrewer` | `Set1` qualitative palette for 5 author colors |
| `viridis` | Imported (available for alternative continuous color scales) |
| `scales` | Axis formatting utilities |

---

## 🚀 How to Run

```r
# Install required packages if needed
install.packages(c("dplyr", "plotly", "ggplot2", "viridis", "RColorBrewer", "scales"))

# Place 'bestsellers with categories.csv' in your working directory
# Open Project_1.Rmd in RStudio and click Run All (or Knit to HTML)
```

---

## 📁 File Structure

```
Walmart-Sales-RShiny/
├── Project_1.Rmd                      # Main R Notebook
├── bestsellers with categories.csv    # Dataset
└── README.md                          # This file
```

---

## 🔑 Key Takeaways

- **Score formula design** — multiplying `User.Rating` by 1,000 before adding `Reviews` ensures that a book rated 4.9 always outranks a 4.8 regardless of review count, but within the same rating tier, review volume is the tiebreaker
- **Deduplication on content, not ID** — books appear across multiple years in the raw data; deduplicating on `(Name, User.Rating, Reviews, Genre)` removes year-repetition while preserving the first known price
- **`free_y` scales in facets** — without `scales = "free_y"` all facets would share one x-axis, making low-score authors' bars nearly invisible against high-score authors
- **Plotly tooltip engineering** — the custom `text` aesthetic in `aes()` with HTML `<br>` line breaks gives the interactive tooltip full book name (not the truncated label), price with dollar sign, and formatted score — information not visible in the static chart
- **Strip text removal** — removing `strip.text.y` keeps the facet panels clean since the fill color and legend already communicate author identity

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
