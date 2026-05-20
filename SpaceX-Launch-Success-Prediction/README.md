# 🚀 SpaceX Falcon 9 First Stage Landing Prediction

> An end-to-end data science capstone project predicting whether the **Falcon 9 first stage will land successfully** — enabling cost estimation of rocket launches. SpaceX advertises Falcon 9 launches at **$62 million** vs. competitors at $165 million+, with much of the savings from first-stage reuse. The project spans API data collection, web scraping, SQL analysis, interactive Folium maps, a Plotly Dash dashboard, and 4 ML classification models.

---

## 📌 Problem Statement

The cost of a Falcon 9 launch depends heavily on whether the first stage lands successfully and can be reused. If we can predict first-stage landing success from launch parameters, we can anticipate launch cost — giving alternate companies a data-driven edge when bidding against SpaceX.

**Target:** Binary classification — did the first stage land successfully? (`1` = success, `0` = failure)

---

## 🗂️ Project Pipeline

```
Lab 1: Data Collection (SpaceX API + Static JSON)
         ↓
Lab 2: Data Wrangling (landing outcome → Class label)
         ↓
Lab 3: SQL Analysis (10 queries on SQLite)
         ↓
Lab 4: Folium Interactive Maps (launch site geography)
         ↓
Lab 5: ML Prediction (4 models + GridSearchCV)
         ↓
Plotly Dash Dashboard (interactive EDA)
```

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | SpaceX REST API (`https://api.spacexdata.com/v4/launches/past`) + static JSON + Wikipedia web scraping |
| Final dataset | **90 Falcon 9 launches** (Falcon 1 launches filtered out) |
| Date range | 2010-06-04 to 2020-11-13 |
| Target | `Class` — 1 (success) / 0 (failure) |
| Overall success rate | **66.7%** in dataset |
| Missing values | PayloadMass (5 NaN → filled with column mean: ~**6,104.95 kg**) |

**Launch Sites:**

| Site | Launches |
|------|---------|
| CCAFS SLC 40 | 55 |
| KSC LC 39A | 22 |
| VAFB SLC 4E | 13 |

---

## 🧪 Lab 1 — Data Collection (SpaceX REST API)

**Endpoint:** `https://api.spacexdata.com/v4/launches/past`

Four helper functions called the API iteratively to resolve IDs into human-readable values:

| Function | Data Extracted |
|----------|---------------|
| `getBoosterVersion(data)` | Booster name from `rocket` ID |
| `getLaunchSite(data)` | Site name, longitude, latitude from `launchpad` ID |
| `getPayloadData(data)` | Payload mass (kg) and orbit from `payloads` ID |
| `getCoreData(data)` | Landing outcome, landing type, flights, grid fins, reuse, legs, landing pad, block, reuse count, serial from `cores` |

**Preprocessing:**
- Filtered to **Falcon 9 only** (removed Falcon 1 entries)
- Removed rows with multiple cores or multiple payloads
- Reset `FlightNumber` to 1–90 sequentially
- `PayloadMass` NaN filled with mean (~6,104.95 kg)
- Saved as `dataset_part_1.csv`

---

## 🔧 Lab 2 — Data Wrangling & Label Creation

**Landing outcome categories and their classification:**

| Outcome | Class |
|---------|-------|
| True ASDS | 1 (success) |
| True RTLS | 1 (success) |
| True Ocean | 1 (success) |
| False ASDS | 0 (failure) |
| False RTLS | 0 (failure) |
| False Ocean | 0 (failure) |
| None ASDS | 0 (failure) |
| None None | 0 (failure) |

**Distribution of outcomes:**
- True ASDS: 41 · None None: 19 · True RTLS: 14 · False ASDS: 6 · True Ocean: 5 · False Ocean: 2 · None ASDS: 2 · False RTLS: 1

**Orbit types analyzed:**

| Orbit | Count |
|-------|-------|
| GTO | 27 |
| ISS | 21 |
| VLEO | 14 |
| PO | 9 |
| LEO | 7 |
| SSO | 5 |
| MEO | 3 |
| HEO, ES-L1, SO, GEO | 1 each |

- Saved as `dataset_part_2.csv`

---

## 🗄️ Lab 3 — SQL Analysis (SQLite)

**Database:** `SPACEXTABLE` loaded via SQLite from `Spacex.csv`

**10 SQL queries with key findings:**

| # | Query | Result |
|---|-------|--------|
| 1 | Unique launch sites | CCAFS LC-40, VAFB SLC-4E, KSC LC-39A, CCAFS SLC-40 |
| 2 | 5 records where Launch_Site LIKE 'CCA%' | First 5 CCAFS records |
| 3 | Total payload by NASA (CRS) | **45,596 kg** |
| 4 | Avg payload of F9 v1.1 booster | **2,928.4 kg** |
| 5 | First successful ground pad landing | **2015-12-22** |
| 6 | Boosters with drone ship success, payload 4,000–6,000 kg | F9 FT B1022, B1026, B1021.2, B1031.2 |
| 7 | Total successful missions | **98 Success** |
| 8 | Boosters with max payload (subquery) | 12 Block 5 variants at **15,600 kg** |
| 9 | 2015 drone ship failures by month | January (B1012, CCAFS LC-40) · April (B1015, CCAFS LC-40) |
| 10 | Ranked landing outcomes 2010–2017 | No attempt (10), Success drone ship (5), Failure drone ship (5) |

**Key SQL techniques:** `DISTINCT`, `LIKE`, `SUM`, `AVG`, `MIN`, `COUNT`, `GROUP BY`, `ORDER BY DESC`, `BETWEEN`, `WHERE` with subquery (`SELECT max(...)`)

---

## 🗺️ Lab 4 — Interactive Folium Maps

**4 launch sites plotted** with MarkerCluster:

| Launch Site | Lat | Long |
|-------------|-----|------|
| CCAFS LC-40 | 28.562302 | -80.577356 |
| CCAFS SLC-40 | 28.563197 | -80.576820 |
| KSC LC-39A | 28.573255 | -80.646895 |
| VAFB SLC-4E | 34.632834 | -120.610745 |

**Visualizations:**
- `folium.Circle` + `folium.Marker` with `DivIcon` labels for each launch site
- `MarkerCluster` with **green markers** (Class=1, success) and **red markers** (Class=0, failure)
- `MousePosition` plugin for real-time coordinate tracking
- `PolyLine` with distance annotations to nearest proximity points

**Distance calculations (Haversine formula):**

| KSC LC-39A proximity | Distance |
|---------------------|---------|
| Nearest coastline | **7.36 km** |
| Nearest city | **72 km** |

**Geographic insights:**
- All launch sites are within close proximity to the coast
- All sites are near the equator (CCAFS/KSC) or at mid-latitude (VAFB) for polar orbits
- Launch sites are set back from cities but close to the shoreline for safety

---

## 📈 EDA Visualizations (Plotly Dash + Matplotlib/Seaborn)

**Key findings from EDA:**

- **CCAFS SLC 40** had the most launches and the most failures — but also the most successes
- **Low payload mass (< 6,000 kg)** correlates with higher success rates
- **ES-L1, GEO, HEO, SSO** orbits showed a success rate of **1.0** (100%)
- Success rate increased year-over-year, with **2019** being the most successful year
- GTO payloads range from 3,800 to 8,000 kg; ISS payloads are lower

**Plotly Dash Dashboard:**
- Pie chart filtered by launch site showing success/failure breakdown
- KSC LC-39A: **76.9% success** rate (highest of all sites)
- Payload range **2,500–7,500 kg** at KSC LC-39A: success rate jumps to **64.3%**
- FT booster version showed the most promising results
- Scatter plot of payload mass vs. outcome with adjustable payload range slider

---

## 🤖 Lab 5 — ML Prediction (Classification)

### Feature Engineering

- **83 columns** after one-hot encoding of categorical variables (orbit, launch site, serial, grid fins, reused, legs, etc.)
- `StandardScaler` applied to all features before model training
- **80/20 train-test split** — `random_state=2`, 18 test samples

### Models & GridSearchCV (cv=10)

**1. Logistic Regression**

| Parameter | Best Value |
|-----------|-----------|
| `C` | **0.01** |
| `penalty` | l2 |
| `solver` | lbfgs |
| CV Accuracy | **84.6%** |
| Test Accuracy | **83.3%** |

**2. Support Vector Machine (SVC)**

| Parameter | Best Value |
|-----------|-----------|
| `C` | **1.0** |
| `gamma` | **0.032** |
| `kernel` | **sigmoid** |
| CV Accuracy | **84.8%** |
| Test Accuracy | **83.3%** |

**3. Decision Tree** ⭐ Best Model

| Parameter | Best Value |
|-----------|-----------|
| `criterion` | gini |
| `max_depth` | **10** |
| `splitter` | random |
| `max_features` | sqrt |
| CV Accuracy | **87.5%** |
| Test Accuracy | **88.9%** ← Highest |

**4. K-Nearest Neighbors (KNN)**

| Parameter | Best Value |
|-----------|-----------|
| `n_neighbors` | **10** |
| `algorithm` | auto |
| `p` | 1 (Manhattan) |
| CV Accuracy | **84.8%** |
| Test Accuracy | **83.3%** |

---

### 📈 Full Model Comparison

| Model | CV Accuracy | Test Accuracy |
|-------|------------|--------------|
| Logistic Regression | 84.6% | 83.3% |
| SVM | 84.8% | 83.3% |
| **Decision Tree** | **87.5%** | **88.9%** |
| KNN | 84.8% | 83.3% |

---

## 🏆 Conclusions

- **Decision Tree** is the best performing model — **88.9% test accuracy**, **87.5% CV accuracy**
- All other models (LR, SVM, KNN) produced identical test accuracy of **83.3%** — a sign the 18-sample test set is a constraint; the confusion matrices reveal the same misclassification pattern (3 false positives)
- **KSC LC-39A** has the highest success rate of all launch sites
- **Payload between 4,000–6,000 kg** is the ideal range for successful launches
- Success rate has consistently climbed year over year, peaking in **2019**

---

## 🛠️ Tools & Libraries

| Category | Tools |
|----------|-------|
| **Language** | Python 3 |
| **Data Collection** | `requests`, `BeautifulSoup4` (web scraping), SpaceX REST API |
| **Data Wrangling** | `pandas`, `numpy` |
| **SQL** | `SQLite3`, `ipython-sql`, `sqlalchemy` |
| **Visualization** | `matplotlib`, `seaborn`, `plotly`, `plotly-dash` |
| **Geospatial** | `folium`, `folium.plugins` (MarkerCluster, MousePosition), Haversine formula |
| **ML** | `scikit-learn` (LogisticRegression, SVC, DecisionTreeClassifier, KNeighborsClassifier, GridSearchCV, StandardScaler, train_test_split) |
| **Environment** | Jupyter Notebook |

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd SpaceX-Launch-Success-Prediction
pip install -r requirements.txt
jupyter notebook
```

**Run labs in order:**
```
Lab_1_Data_Collection_API.ipynb
Lab_2_Data_Wrangling.ipynb
Lab_3_SQL_Analysis.ipynb
Lab_4_Folium_Maps.ipynb
Lab_5_ML_Prediction.ipynb
```

**Required packages:**
```
requests beautifulsoup4 pandas numpy matplotlib seaborn
plotly dash folium scikit-learn sqlalchemy ipython-sql
```

---

## 📁 File Structure

```
SpaceX-Launch-Success-Prediction/
├── Lab_1_Data_Collection_API.ipynb      # SpaceX API + data extraction
├── Lab_2_Data_Wrangling.ipynb           # Landing outcome → Class label
├── Lab_3_SQL_Analysis.ipynb             # 10 SQL queries on SQLite
├── Lab_4_Folium_Maps.ipynb              # Interactive launch site maps
├── Lab_5_ML_Prediction.ipynb            # 4 ML models + GridSearchCV
├── dataset_part_1.csv                   # Raw Falcon 9 dataset (90 rows)
├── dataset_part_2.csv                   # Dataset with Class label
├── dataset_part_3.csv                   # One-hot encoded features (83 cols)
├── my_data1.db                          # SQLite database
├── spacex_web_scraped.csv               # Wikipedia scraped data
├── Project_report.pptx                  # Final presentation
└── README.md                            # This file
```

---

## 🔑 Key Takeaways

- **API + web scraping together** — combining the SpaceX REST API (structured data) with Wikipedia BeautifulSoup scraping (historical records) gave a richer dataset than either source alone
- **Helper function pattern** — using `getBoosterVersion()`, `getLaunchSite()`, `getPayloadData()`, `getCoreData()` to resolve API IDs into readable values is a clean, production-grade ETL pattern
- **83 features after encoding** — one-hot encoding of boosters, sites, serials, and boolean features expands the feature space significantly; `StandardScaler` is essential before any distance-based model
- **18 test samples** — the small test set means all models producing 83.3% accuracy (15/18 correct) tells the same story; the 3 false positives are the same 3 samples misclassified across LR, SVM, and KNN
- **Decision Tree at depth=10** found the optimal complexity — shallow enough to generalise, deep enough to capture non-linear patterns in orbit type, payload mass, and booster reuse count
- **Geography matters** — all launch sites hug the coastline (< 10 km) for debris safety, and east-coast sites (CCAFS, KSC) leverage Earth's rotational speed for eastward launches; VAFB handles polar orbits
- **KSC LC-39A payload sweet spot** — payload range 2,500–7,500 kg at Kennedy pushes success from 43.7% (all payloads) to 64.3%, a concrete operational insight for mission planners
- **Year-over-year improvement** — the climbing success rate reflects SpaceX's iterative learning: better grid fins, landing leg design, ASDS positioning, and Block 5 booster standardisation all compound over time

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
