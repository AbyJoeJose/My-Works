# 🎬 IMDB Movie Analysis (SQL)

> A comprehensive SQL-based exploratory analysis of the IMDB movie database across 6 relational tables — answering 29 structured business questions for RSVP Movies to guide their next film project using data-driven insights on genres, ratings, directors, actors, and production companies.

---

## 📌 Problem Statement

RSVP Movies, an Indian production house, is planning to release a global film in 2022. Before committing to a project, they need data-driven answers about what makes movies successful — which genres perform best, which directors and actors deliver consistently high ratings, and which production houses are the most credible partners. This project answers 29 analytical questions using SQL across a multi-table IMDB dataset.

---

## 📊 Dataset

| Table | Description |
|-------|-------------|
| `movie` | Core movie metadata — title, year, date, duration, country, language, production company, gross income |
| `genre` | Genre tags per movie (one movie can have multiple genres) |
| `ratings` | Average rating, median rating, total votes per movie |
| `names` | Person details — actors, directors, name IDs |
| `role_mapping` | Maps names to movies with category (actor/actress/director) |
| `director_mapping` | Maps director name IDs to movie IDs |

**Database:** MySQL (`USE imdb`)

---

## 🧠 Analysis Structure

### Segment 1: Movie & Genre Tables

**Q1 — Row counts per table**
Verified dataset size across all 6 tables.

**Q2 — Null values in movie table**
4 columns contain nulls: `country`, `worlwide_gross_income`, `languages`, `production_company`. `id` and `title` are fully populated.

**Q3 — Movies released per year and per month**
- Yearly trend: 2017–2019 breakdown
- Monthly trend: **March has the highest number of movie releases**

**Q4 — USA and India production in 2019**
Both USA and India produced over 1,000 movies in 2019 combined.

**Q5 — Unique genres**
Listed all distinct genres present in the dataset.

**Q6 — Genre with highest movie count**
**Drama** has the highest number of movies overall.

**Q7 — Movies with only one genre**
Using a CTE with `HAVING count(genre) = 1` — over **3,000 movies** belong to exactly one genre.

**Q8 — Average duration per genre**
Two approaches: single-genre movies only, and all movies (including multi-genre).
- Drama average duration: **106.77 minutes**

**Q9 — Thriller genre rank**
Using `DENSE_RANK()` ordered by movie count:
- **Thriller ranks in the top 3** among all genres by number of movies.

---

### Segment 2: Ratings Table

**Q10 — Min/Max of ratings columns**
No outliers found. All values within expected ranges across `avg_rating`, `median_rating`, and `total_votes`.

**Q11 — Top 10 movies by average rating**
Using `DENSE_RANK() OVER (ORDER BY avg_rating DESC)` joined with movie titles.

**Q12 — Movie count by median rating**
`median_rating = 7` has the **highest number of movies**.

**Q13 — Production house with most hit movies (avg_rating > 8)**
Using `DENSE_RANK()` on hit movie count:
**Dream Warrior Pictures** or **National Theatre Live** rank at the top (both tied at #1).

**Q14 — Genre-wise movies in USA, March 2017, votes > 1,000**
Multi-join query filtering on `year=2017`, `month=3`, `country='USA'`, `total_votes>1000` grouped by genre.

**Q15 — Movies starting with 'The' and avg_rating > 8**
Using `LIKE 'The%'` filter with `avg_rating > 8` — results ordered by rating descending with genre included.

**Q16 — Movies between Apr 2018 – Apr 2019 with median rating = 8**
Date range filter: `BETWEEN '2018-04-01' AND '2019-04-01'` with `median_rating = 8`.

**Q17 — German vs Italian movies: total votes**
Filtered on `languages = 'German'` or `'Italian'` (exact language match):
**German movies received more total votes than Italian movies.**

---

### Segment 3: Names, Directors & Actors

**Q18 — Null values in names table**
`name` has no nulls. `height`, `date_of_birth`, and `known_for_movies` have varying null counts.

**Q19 — Top 3 directors in top 3 genres (avg_rating > 8)**
Two-CTE approach:
1. Find top 3 genres by movie count where `avg_rating > 8`
2. Find directors within those genres with the most high-rated movies

**Result: James Mangold** is the top director (known for Logan, The Wolverine).

**Q20 — Top 2 actors with median_rating >= 8**
Filtered on `median_rating > 8` (note: strict greater than, not >=):
**Mohanlal** appears in the top results.

**Q21 — Top 3 production houses by total votes**
Using `DENSE_RANK() OVER (ORDER BY SUM(total_votes) DESC)`:
**Marvel Studios** leads — rules the movie world by vote count.

**Q22 — Indian actors ranked by weighted average rating (min 5 movies)**
Weighted average formula: `SUM(avg_rating × total_votes) / SUM(total_votes)`
Filtered on `country = 'India'`, minimum 5 movies:
**Vijay Sethupathi** is ranked #1.

**Q23 — Top 5 Hindi actresses in India (min 3 movies)**
Filtered on `category = 'Actress'`, `country = 'India'`, `languages = 'Hindi'`, minimum 3 movies:
**Taapsee Pannu** tops with weighted average rating **7.74**.

**Q24 — Thriller movies classified by avg_rating**
CASE WHEN classification:

| Rating | Category |
|--------|----------|
| > 8 | Superhit Movies |
| 7–8 | Hit Movies |
| 5–7 | One-time-watch movies |
| < 5 | Flop movies |

---

### Segment 4: Advanced Analytics

**Q25 — Genre-wise running total and moving average of duration**
Using named window functions:
- `ROWS UNBOUNDED PRECEDING` for running total
- `ROWS 6 PRECEDING` for 6-row moving average
Both ordered by `avg(duration)`.

**Q26 — Top 5 highest-grossing movies per year in top 3 genres**
Two-CTE approach:
1. Find top 3 genres by total movie count
2. Rank movies within each year by `worlwide_gross_income` using `DENSE_RANK() OVER (PARTITION BY year ORDER BY worlwide_gross_income DESC)`

Filter: `movie_rank <= 5`

**Q27 — Top 2 production houses: multilingual hits (median_rating >= 8)**
Multilingual detection: `POSITION(',' IN languages) > 0` (comma in language field = multiple languages).
Filtered on `median_rating >= 8` and non-null production company.

**Q28 — Top 3 actresses in Drama superhit movies (avg_rating > 8)**
Filtered on `avg_rating > 8`, `genre = 'Drama'`, `category = 'Actress'`.
Ranked by `avg(avg_rating)` using `DENSE_RANK()`.
Result: **Laura Dern** at rank 1.

**Q29 — Top 9 directors: full performance profile**
CTE computes inter-movie gaps using `LEAD(date_published)` partitioned by director.
Final query joins back to get per-director aggregates:

| Column | Calculation |
|--------|-------------|
| `number_of_movies` | COUNT |
| `avg_inter_movie_days` | AVG of DATEDIFF between consecutive releases |
| `avg_rating` | AVG of avg_rating |
| `total_votes` | SUM |
| `min_rating` | MIN |
| `max_rating` | MAX |
| `total_duration` | SUM |

**Top director: A.L. Vijay** (5 movies, avg inter-movie gap = 177 days, avg rating = 5.65)

---

## 🔑 Key Business Recommendations for RSVP Movies

| Decision | Insight |
|----------|---------|
| **Genre** | Focus on **Drama** — highest movie count and solid average duration (~107 min) |
| **Director** | Hire **James Mangold** — top director across top 3 high-rated genres |
| **Lead Actor** | Cast **Vijay Sethupathi** — highest weighted rating among Indian actors (5+ movies) |
| **Lead Actress** | Cast **Taapsee Pannu** — highest weighted rating among Hindi Indian actresses |
| **Production Partner** | Partner with **Dream Warrior Pictures** or **National Theatre Live** for hit movies |
| **Global Partner** | Collaborate with **Marvel Studios** — highest total vote count globally |
| **Release Month** | Avoid March (overcrowded); consider less competitive months |

---

## 🛠️ SQL Techniques Used

| Technique | Where Used |
|-----------|-----------|
| `DENSE_RANK() OVER (ORDER BY ...)` | Movie rankings, genre ranks, actor/director rankings |
| `PARTITION BY` | Year-wise movie rankings (Q26), director inter-movie gaps (Q29) |
| `LEAD()` window function | Inter-movie duration calculation (Q29) |
| `DATEDIFF()` | Days between consecutive director releases |
| `ROWS UNBOUNDED PRECEDING` | Running total of genre duration (Q25) |
| `ROWS 6 PRECEDING` | Moving average of genre duration (Q25) |
| `WITH` (CTEs) | Top genres (Q19, Q26), inter-movie days (Q29) |
| `CASE WHEN` | Thriller movie classification (Q24) |
| `POSITION(',' IN languages)` | Multilingual movie detection (Q27) |
| Weighted average | `SUM(avg_rating × total_votes) / SUM(total_votes)` for actors/actresses |
| `HAVING` | Minimum movie count filters |
| Multi-table `INNER JOIN` | All queries across 6 tables |

---

## 🛠️ Tools

**Language:** SQL · **Database:** MySQL · **Schema:** `imdb`

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd IMDB-Movie-Analysis-SQL
```

```sql
-- In your MySQL client:
CREATE DATABASE imdb;
USE imdb;

-- Import the tables (movie, genre, ratings, names, role_mapping, director_mapping)
-- Then run the SQL script:
SOURCE IMDB_Movie_Analysis.sql;
```

---

## 📁 File Structure

```
IMDB-Movie-Analysis-SQL/
├── IMDB_Movie_Analysis.sql     # Complete SQL script (29 questions)
└── README.md                   # This file
```

---

## 🔑 Key Takeaways

- **Drama is the dominant genre** — most movies, solid ratings, and ~107 min average duration make it the safest bet for RSVP's next project
- **Weighted average ratings** (votes × rating / total votes) provide a more reliable quality signal than simple average ratings
- **Thriller ranks top 3 by volume** — a viable genre option with a strong audience
- **`POSITION(',' IN languages) > 0`** is an elegant approach to detect multilingual movies without needing a separate join
- **LEAD() + DATEDIFF()** within a CTE provides a clean way to compute inter-movie intervals for director productivity analysis
- **German movies outperform Italian** in total vote engagement — suggesting stronger global audience reach
- **Median rating of 7** is the most common — the dataset skews toward moderately well-rated films

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
