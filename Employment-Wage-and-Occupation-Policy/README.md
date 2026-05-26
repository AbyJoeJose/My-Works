# 📊 Employment, Wage & Occupation Policy Analysis
### Workforce Research · Compensation Analytics · Statistical Inference · Compliance Detection

> **A quantitative policy research project** applying population-level employment analysis, regression modeling, non-parametric hypothesis testing, wage inequality measurement, and machine-learning-based anomaly detection to New York State's Occupational Employment and Wage Statistics — producing defensible, data-driven findings for labor policy, workforce enforcement, and regional economic development.

---

## 📌 Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Analytical Framework](#analytical-framework)
- [Project Workflow](#project-workflow)
- [Key Techniques & Methods](#key-techniques--methods)
- [Statistical Results](#statistical-results)
- [Policy Findings](#policy-findings)
- [Visualizations](#visualizations)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)

---

## 🎯 Project Overview

This project simulates a **quantitative policy research engagement** — the kind produced by labor economics units, workforce analytics teams, and policy research organizations. The goal is to move beyond descriptive statistics and produce **statistically validated, enforcement-relevant, and policy-actionable findings** from public workforce data.

The analysis answers four core research questions:

| # | Research Question | Method |
|---|------------------|--------|
| 1 | What is the population-level structure of NYS employment across sectors and regions? | EDA · Employment distribution analysis · Heatmaps |
| 2 | What factors statistically determine wage levels — and how much does region vs. occupation explain? | OLS Multiple Regression · Coefficient analysis |
| 3 | Are observed wage differences between regions statistically significant, or within sampling noise? | Mann-Whitney U · Kruskal-Wallis · Post-hoc pairwise testing |
| 4 | Which occupations show anomalous wage structures consistent with compliance risk or under-compensation? | Isolation Forest · Z-score detection · IQR flagging · CV analysis |

---

## 📂 Dataset

**Source:** New York State Department of Labor — Occupational Employment and Wage Statistics (OEWS)

| Attribute | Detail |
|-----------|--------|
| **Total records** | 7,059 rows |
| **Geographic scope** | 1 statewide total + 10 sub-regions |
| **Occupation coverage** | 23 major SOC groups · ~650 detailed occupations |
| **Wage variables** | Entry Wage · Mean Wage · Median Wage · Experienced Wage |
| **Employment variable** | Total workers per occupation-region combination |
| **Classification system** | Standard Occupational Classification (SOC) codes |

**Key columns:**

| Column | Description |
|--------|-------------|
| `Area Name` | One of 11 NYS geographic areas (statewide or regional) |
| `Standard Occupational Code` | SOC code (e.g., `29-1215` = Family Medicine Physicians) |
| `Occupational Title` | Full occupation name |
| `Employment` | Estimated number of workers |
| `Mean Wage` | Annual mean wage (USD) |
| `Median Wage` | Annual median wage (USD) |
| `Entry Wage` | 25th percentile wage proxy — new entrant compensation |
| `Experienced Wage` | 75th percentile wage proxy — career earnings |

---

## 🧠 Analytical Framework

This project is structured around four interconnected analytical pillars — each producing findings with direct policy relevance:

```
┌─────────────────────────────────────────────────────────────────────┐
│  PILLAR 1: Population-Level Employment Analysis                     │
│  → Who works where? What sectors dominate? How does occupation      │
│    mix vary across regions?                                         │
├─────────────────────────────────────────────────────────────────────┤
│  PILLAR 2: Wage Distribution Modeling & Inequality Measurement      │
│  → What is the shape of wage distributions? Where is inequality     │
│    concentrated? Which sectors have the highest high-earner pull?   │
├─────────────────────────────────────────────────────────────────────┤
│  PILLAR 3: Regression & Hypothesis Testing                          │
│  → What statistically drives wages? Are regional differences real   │
│    or confounded by occupation mix? Which pairs differ?             │
├─────────────────────────────────────────────────────────────────────┤
│  PILLAR 4: Anomaly Detection & Compliance Assessment                │
│  → Which occupations have anomalous wage structures? Where are      │
│    near-floor concentrations? What signals compliance risk?         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Project Workflow

```
Raw OEWS Data (7,059 rows)
         │
         ▼
  Data Cleaning & Preprocessing
  ├── Separate statewide vs. regional (Area Type 1 vs. 10)
  ├── Filter aggregate rows (SOC 00-0000)
  ├── Group-median imputation for missing wages & employment
  ├── Convert days → years for employment duration fields
  └── Engineer: Wage_Gap · Mean_Median_Spread · Wage_Ratio · Log_Employment
         │
         ▼
  EDA — Employment Landscape                     [Section 3]
  ├── Employment by major occupation group (statewide)
  ├── Regional employment distribution (bar + pie)
  └── Occupation-mix heatmap by region (normalized %)
         │
         ▼
  Regional Wage Analysis                          [Section 4]
  ├── Mean wage by region vs. NYS benchmark
  ├── Entry vs. Experienced wage ladder (career progression)
  ├── Top-paying occupations per region
  └── Career wage gap heatmap (region × occupation group)
         │
         ▼
  Wage Distribution Modeling                      [Section 5]
  ├── Distribution histograms + normal curve fits (3 wage tiers)
  ├── Gini coefficient per occupation group
  ├── Box plots — within-group wage spread
  └── Mean–Median spread (skewness / high-earner pull signal)
         │
         ▼
  OLS Regression — Wage Determinants              [Section 6]
  ├── 5-predictor OLS model (R² = 0.974)
  ├── Coefficient plot with 95% CIs
  └── Actual vs. Predicted + Residual diagnostics
         │
         ▼
  Statistical Hypothesis Testing                  [Section 7]
  ├── D'Agostino-Pearson normality tests (all wages)
  ├── Mann-Whitney U: NYC vs. Rest-of-State
  ├── Kruskal-Wallis: All 10 regions simultaneously
  └── Post-hoc pairwise comparisons: 45 region pairs
         │
         ▼
  Anomaly Detection                               [Section 8]
  ├── Z-score flagging (|Z| > 2.5 on mean wages)
  ├── Isolation Forest — multivariate anomalies (5% contamination)
  ├── IQR-based wage suppression detection (below Q1 within peer group)
  └── Anomaly visualization: wage-space scatter + score distribution
         │
         ▼
  Compliance Pattern Assessment                   [Section 9]
  ├── Entry wage vs. statutory minimum (NYC: $33,280 / Rest: $31,200)
  ├── Compliance flag heatmap (region × occupation group)
  ├── Flat career progression flagging (<20% entry-to-experienced growth)
  └── CV-based regional pay equity analysis (CV > 20% = inequity flag)
         │
         ▼
  Policy Findings & Executive Dashboard           [Section 10]
  └── 7 data-driven findings + stakeholder recommendations
```

---

## 🔬 Key Techniques & Methods

### Data Engineering
- **Group-median imputation** by Occupational Title for missing wages — preserves distributional properties within peer groups rather than applying a single global fill
- **Log-transformation of Employment** (`log1p`) to normalize the right-skewed employment variable for regression
- **Derived features:** `Wage_Gap` (Experienced − Entry), `Mean_Median_Spread`, `Wage_Ratio`, `Progression_Ratio` — each engineered to capture a distinct compensation signal

### Inequality Measurement
- **Gini Coefficient** computed per occupation group using the exact formula, revealing that Healthcare Practitioners (Gini = 0.323) and Transportation (0.281) are the most internally unequal groups
- **Mean–Median Spread** as a practical skewness proxy — Healthcare Practitioners ($28,525 spread) and Management ($20,398) show the strongest high-earner concentration effects
- **Coefficient of Variation (CV)** computed across regions per occupation — 70 occupations exceed the 20% inequity threshold

### Regression Analysis
- **OLS Multiple Regression** (statsmodels) with 5 predictors on 6,025 observations — R² = 0.974, F = 45,656 (p ≈ 0)
- **Key finding from coefficients:** `Region_Code` is **not statistically significant** (p = 0.333) once occupation group is controlled, but `SOC_Major_Code` (p ≈ 0, coef = 199) and `Entry Wage` (p ≈ 0, coef = 1.233) are dominant — proving that *what you do* matters more than *where you live* for wage determination in NYS
- **Residual diagnostics** confirm heteroscedasticity at the high-wage tail — structurally consistent with the known non-normality of wage distributions

### Non-Parametric Hypothesis Testing
All wage distributions failed normality (D'Agostino-Pearson p < 10⁻⁸⁵), making **non-parametric tests the methodologically correct choice**:

| Test | Purpose | Result |
|------|---------|--------|
| **Mann-Whitney U** | NYC vs. Rest-of-State wage comparison | p = 3.6×10⁻²⁴ — highly significant |
| **Kruskal-Wallis H** | Simultaneous test across all 10 regions | H = 262.66, p = 2.1×10⁻⁵¹ |
| **Post-hoc Pairwise MWU** | Which specific region pairs differ? | 31 of 45 pairs (69%) significant |
| **Effect size (η²)** | Practical significance of regional differences | η² = 0.041 (small-medium) |

### Anomaly Detection
- **Z-score method** (|Z| > 2.5): Flagged 25 records (3.2%) — top anomalies are high-earning medical specialists (Pediatric Surgeons: Z = 6.42, mean = $412,754)
- **Isolation Forest** (scikit-learn, 200 trees, 5% contamination): Multivariate anomaly detection across 5 wage features simultaneously — catches occupations with anomalous *combinations* of wage metrics, not just outliers on a single dimension
- **IQR peer-group flagging:** 197 occupations flagged as potentially wage-suppressed (below Q1 of their SOC major group) — from Shampooers ($33,784) to Office Support Workers ($37,219)

### Compliance Analytics
- **Wage floor benchmarking** against NYS statutory minimums with regional differentiation (NYC vs. Rest-of-State floors)
- **Career progression ratio** (Experienced ÷ Entry) — 283 occupation-region combinations show <20% career wage growth, identifying structural career stagnation
- **CV-based equity flagging** — 70 occupations exceed 20% regional wage variation, with News Analysts (CV = 88.4%) and Financial Examiners (CV = 28.4%) as extreme cases

---

## 📈 Statistical Results

### OLS Regression — Full Output

```
Dep. Variable:    Mean Wage     R-squared:      0.974
Model:            OLS           Adj. R-squared: 0.974
No. Observations: 6,025         F-statistic:    45,656
                                Prob (F-stat):  ≈ 0.00

Predictors:
────────────────────────────────────────────────────────────
Variable          Coef      Std Err    t        P>|t|
────────────────────────────────────────────────────────────
const           -11,350     593.4    -19.12    0.000 ***
Entry Wage        1.233      0.005   245.14    0.000 ***
Wage_Gap          0.614      0.003   182.47    0.000 ***
Log_Employment  -159.27     72.97    -2.18     0.029 *
Region_Code       34.70     35.87     0.97     0.333     ← NOT significant
SOC_Major_Code   199.19     17.18    11.60     0.000 ***
────────────────────────────────────────────────────────────
*** p < 0.001   * p < 0.05
```

### Hypothesis Testing Summary

```
Test                          Statistic    p-value       Conclusion
─────────────────────────────────────────────────────────────────────────
Normality — Entry Wage        392.58       5.65×10⁻⁸⁶   NOT normal
Normality — Mean Wage         417.19       2.55×10⁻⁹¹   NOT normal
Normality — Experienced Wage  391.18       1.14×10⁻⁸⁵   NOT normal
Mann-Whitney (NYC vs RoS)     U=2,393,866  3.63×10⁻²⁴   Significant gap
Kruskal-Wallis (10 regions)   H=262.66     2.11×10⁻⁵¹   Significant variation
Post-hoc significant pairs    31 of 45     —             69% differ
```

### Top Anomalies — Z-Score Detection

| Occupation | Mean Wage | Z-Score | Wage Gap |
|-----------|-----------|---------|----------|
| Pediatric Surgeons | $412,754 | 6.42 | −$166,834 |
| Cardiologists | $391,687 | 6.00 | +$282,100 |
| Ophthalmologists | $358,083 | 5.34 | +$187,910 |
| Chief Executives | $349,390 | 5.16 | +$312,476 |
| Radiologists | $321,645 | 4.61 | +$355,565 |

### Wage Inequality by Occupation Group (Gini)

| Occupation Group | Gini Coefficient | Interpretation |
|-----------------|-----------------|----------------|
| Healthcare Practitioners | 0.323 | Highest inequality — surgeons to aides |
| Transportation | 0.281 | Pilots vs. parking attendants |
| Sales | 0.267 | Commission-driven dispersion |
| Legal | 0.211 | Partners vs. paralegals |
| Arts & Media | 0.204 | Stars vs. entry-level roles |
| Community/Social Svc | 0.083 | Most equal — compressed wages |

---

## 💡 Policy Findings

### Finding 1 — Wage Inequality Is Structurally Embedded by Occupation, Not Random
All three wage distributions (Entry, Mean, Experienced) are severely right-skewed — rejecting normality at p < 10⁻⁸⁵. The mean exceeds the median by $9K–$29K across wage tiers. Healthcare Practitioners have the highest Gini (0.323) and the largest mean–median spread ($28,525), with Pediatric Surgeons (6.42σ above mean) and Shampooers ($33,784) coexisting in the same occupation group framework.

> **Implication:** Median wages — not means — should be the standard for policy benchmarking. Aggregate mean reporting overstates typical compensation for 75%+ of workers in high-Gini sectors.

### Finding 2 — NYC's Wage Premium Is Statistically Unambiguous; 8 of 10 Regions Earn Below the State Average
NYC mean wage ($97,896) exceeds the Rest-of-State ($80,088) by 22% — confirmed at p = 3.6×10⁻²⁴. Three upstate region pairs (Central NY/Southern Tier, Central NY/Western NY, Central NY/Finger Lakes) are statistically indistinguishable (all p > 0.47), forming a depressed wage cluster earning $67K–$72K.

> **Implication:** The NYC/upstate divide is structural. Regional minimum wage differentiation and economic development investment in the upstate cluster are empirically justified.

### Finding 3 — Occupation Group Determines Wages; Region Does Not, Once Controlled
The OLS model (R² = 0.974) shows `Region_Code` is **not statistically significant** (p = 0.333) after controlling for occupation group. `Entry Wage` (coef = 1.233) and `SOC_Major_Code` (coef = 199) are the dominant predictors. What sector you work in matters far more than where in NYS you live.

> **Implication:** Sectoral wage policies (occupation-tier minimums, sectoral bargaining) will outperform region-only wage interventions. This is the single most actionable finding for policy design.

### Finding 4 — 2.72M Workers Are Concentrated in High-Employment, Low-Wage Sectors
Office/Admin (1.16M), Healthcare Support (0.82M), and Food Service (0.73M) account for 28% of the NYS workforce — all earning below $60K mean wages. Office/Admin alone represents 15–18.5% of every regional workforce.

> **Implication:** Meaningful wage growth for the NYS median worker requires targeted intervention in these three sectors — not broad economy-wide policies.

### Finding 5 — 45% of Occupation-Region Combinations Show Flat Career Progression
283 of ~630 occupation-region combinations show less than 20% wage growth from entry to experienced level. Service workers — Shampooers, Grounds Maintenance Workers, Ushers — show virtually no career ladder (progression ratio ≈ 1.00–1.05).

> **Implication:** Apprenticeship programs, wage-step transparency requirements, and pay-band policies could address stagnation in NYS's largest employment sectors.

### Finding 6 — 703 Records Are Near the Statutory Wage Floor; No Outright Violations Detected
Compliance rate is 88.3%. The 703 "near-floor" observations are concentrated in Personal Care, Food Service, and Retail — with entry wages at exactly $32,240/year across North Country, Western New York, and Mohawk Valley. No records fall below the statutory minimum.

> **Implication:** Near-floor concentration warrants proactive compliance outreach in upstate regions — particularly North Country and Western New York — before any regulatory floor adjustment.

### Finding 7 — 70 Occupations Show Regional Pay Inequity Exceeding 20% CV
News Analysts (CV = 88.4%), Financial Examiners (CV = 28.4%), Anesthesiologists (CV = 28.4%), and Skincare Specialists (CV = 31.5%) lead the regional inequity list. In regulated sectors, these gaps have direct public-service implications beyond worker equity.

> **Implication:** Wage equity audits should prioritize high-CV occupations in regulated sectors. Regional pay disparity in Financial Examiners and Anesthesiologists signals enforcement capacity and healthcare access risks, respectively.

---

## 📊 Visualizations

The project produces **17 publication-quality visualizations** across the analytical pipeline:

| # | Visualization | Purpose |
|---|--------------|---------|
| 1 | Horizontal bar — Employment by major occupation group | Population-level workforce composition |
| 2 | Bar + pie — Regional employment distribution | Geographic workforce share |
| 3 | Heatmap — Occupation mix by region (normalized %) | Regional sector specialization |
| 4 | Horizontal bar — Mean wage by region vs. NYS benchmark | Regional wage benchmarking |
| 5 | Layered bar — Entry vs. Experienced wage by region | Career earnings progression ladder |
| 6 | Heatmap — Career wage gap by region × occupation group | Sector × geography progression matrix |
| 7 | Triple histogram + normal fits — Entry/Mean/Experienced wages | Distribution shape & normality assessment |
| 8 | Horizontal bar — Gini coefficient by occupation group | Within-group inequality ranking |
| 9 | Box plots — Wage spread by occupation group | IQR dispersion & outlier visualization |
| 10 | Horizontal bar — Mean–Median spread by group | High-earner skewness signal |
| 11 | Coefficient plot with 95% CIs — OLS regression | Statistical significance of wage predictors |
| 12 | Scatter + residual — Actual vs. Predicted wages | Model diagnostics & homoscedasticity check |
| 13 | Overlapping histogram — NYC vs. Rest-of-State wages | Mann-Whitney visual support |
| 14 | Scatter + histogram — Isolation Forest anomaly space | Multivariate anomaly visualization |
| 15 | Heatmap — Compliance flags by region × occupation | Enforcement priority mapping |
| 16 | Histogram — CV distribution with equity threshold | Regional pay disparity overview |
| 17 | 6-panel policy dashboard | Executive summary visualization |

---

## 🛠 Tech Stack

| Category | Tools & Libraries |
|----------|------------------|
| **Data Manipulation** | `pandas` · `numpy` |
| **Statistical Modeling** | `statsmodels` (OLS, residual diagnostics) |
| **Machine Learning** | `scikit-learn` (Isolation Forest, StandardScaler, LabelEncoder) |
| **Statistical Testing** | `scipy.stats` (normaltest, mannwhitneyu, kruskal, shapiro) |
| **Visualization** | `matplotlib` · `seaborn` |
| **Inequality Measurement** | Custom Gini implementation (numpy) |
| **Environment** | Jupyter Notebook · Python 3.10+ |
| **Data Source** | NYS Department of Labor — OEWS Public Dataset |

---

## 📁 Project Structure

```
Employment-Wage-Occupation-Policy/
│
├── Occupational_Employment_Wage_Analysis.ipynb   # Full analysis notebook (10 sections)
├── Occupational_Employment_and_Wage_Statistics.csv  # NYS OEWS dataset
└── README.md                                     # This file
```

**Notebook sections:**

```
Section 1  — Setup & Library Imports
Section 2  — Data Cleaning & Feature Engineering
Section 3  — EDA: Employment Landscape
Section 4  — Regional Wage Analysis
Section 5  — Wage Distribution Modeling & Inequality
Section 6  — OLS Regression Analysis
Section 7  — Statistical Hypothesis Testing
Section 8  — Anomaly Detection
Section 9  — Compliance Pattern Assessment
Section 10 — Policy Findings & Executive Dashboard
```

---

## ▶️ How to Run

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/Employment-Wage-Occupation-Policy.git
   cd Employment-Wage-Occupation-Policy
   ```

2. **Install dependencies**
   ```bash
   pip install pandas numpy matplotlib seaborn scipy statsmodels scikit-learn jupyter
   ```

3. **Launch the notebook**
   ```bash
   jupyter notebook Occupational_Employment_Wage_Analysis.ipynb
   ```

4. **Run all cells sequentially** — each section builds on prior feature engineering. The dataset must be in the same directory as the notebook.

---

## 🤝 Connect

- 💼 [LinkedIn](https://www.linkedin.com/in/aby-joe-jose-88959021b/)
- 📧 abyjoejose00@gmail.com

---

*This project demonstrates applied quantitative policy research methodology — combining statistical rigor, machine learning, and domain-relevant analytical design to produce findings suitable for workforce policy, labor enforcement, and regional economic strategy contexts.*
