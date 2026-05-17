# Aby Joe Jose — Data Science Portfolio
📍 Potsdam, NY &nbsp;|&nbsp; [LinkedIn](https://www.linkedin.com/in/aby-joe-jose-88959021b/) &nbsp;|&nbsp; [Email](mailto:abyjoejose00@gmail.com)
> End-to-end data science projects spanning machine learning, NLP, SQL analytics, deep learning, LLMs, and interactive dashboards — built on real-world datasets.

---

## 📂 Projects

### 🤖 Machine Learning

---

#### [🏠 AirBnB Price Prediction](./AirBnB-Price-Prediction/)
Predicts nightly listing prices for 50,000+ New York City Airbnb properties using regression models. Includes geospatial analysis, feature engineering, and comparison of Linear Regression, Random Forest, and Gradient Boosting.

**Tools:** Python · Pandas · Scikit-learn · Folium · Seaborn  
**Techniques:** Regression · Feature Engineering · Geospatial Analysis

---

#### [🌦️ Australia Weather Prediction](./Australia-Weather-Prediction/)
Binary classifier to predict next-day rainfall across Australia using 145,000+ daily weather station observations. Addresses class imbalance and evaluates models using ROC-AUC and F1.

**Tools:** Python · Pandas · Scikit-learn · XGBoost · Seaborn  
**Techniques:** Classification · Class Imbalance · Ensemble Methods

---

#### [🚲 Bike Sharing Profit Prediction](./Bike-Sharing-Profit-Prediction/)
Forecasts hourly bike rental demand over two years of data to help bike-sharing operators pre-position fleets and maximize revenue. Captures commute-hour peaks and seasonal trends.

**Tools:** Python · Pandas · Scikit-learn · Matplotlib  
**Techniques:** Regression · Time-Series Features · Ensemble Methods

---

#### [🗽 NYC Borough Prediction](./NYC-Borough-Prediction/)
Classifies NYC taxi pickups into the five boroughs using geospatial coordinates, time-of-day, and trip features. Explores geospatial decision boundaries and handles significant class imbalance.

**Tools:** Python · Pandas · Scikit-learn · Folium · Seaborn  
**Techniques:** Multiclass Classification · Geospatial Features · Random Forest

---

#### [📱 Telecom Customer Churn Prediction](./Telecom-Churn-Prediction/)
Identifies telecom customers at risk of cancelling their subscription, enabling proactive retention campaigns. Prioritizes recall to minimize missed churners — the most costly error for the business.

**Tools:** Python · Pandas · Scikit-learn · XGBoost · imbalanced-learn  
**Techniques:** Binary Classification · SMOTE · ROC-AUC · Precision-Recall

---

#### [🚀 SpaceX Falcon 9 Launch Success Prediction](./SpaceX-Launch-Success-Prediction/)
Predicts whether a Falcon 9 first-stage booster will successfully land and be recovered — enabling cost estimation for competing launch companies. Data collected via the SpaceX REST API and web scraping.

**Tools:** Python · Pandas · Scikit-learn · Folium · Plotly · Requests · BeautifulSoup  
**Techniques:** Classification · REST API · Web Scraping · GridSearchCV · Interactive Maps

---

### 🔤 Natural Language Processing

---

#### [🔤 NLP Pipeline: Job Market Analysis](./NLP-Job-Market-Analysis/)
End-to-end NLP pipeline applied to 50,000+ real job postings — progressing from classical NLP through deep learning and production LLM systems.

| Module | Technique |
|--------|-----------|
| Text Classification | CountVectorizer + Logistic Regression (16 scenarios) |
| Tokenization | NLTK, BPE from scratch, Herdan's Law |
| Word Vectors | TF-IDF, GloVe, Word2Vec, cosine similarity |
| Neural Networks | PyTorch feedforward network on TF-IDF features |
| Language Generation | Fine-tuned DistilGPT2, beam search, top-p/top-k |
| NER / Skill Tagging | Fine-tuned DistilBERT (BIO scheme) |
| Prompt Engineering | Mistral-7B on AWS Bedrock — structured JSON extraction |
| RAG System | FAISS + sentence-transformers + Mistral-7B + LangChain |

**Tools:** Python · NLTK · HuggingFace Transformers · PyTorch · Gensim · LangChain · AWS Bedrock · FAISS  
**Techniques:** Text Classification · NER · Language Modeling · Semantic Search · RAG · Prompt Engineering

---

### 🗄️ SQL & Data Engineering

---

#### [🎬 IMDB Movie Analysis (SQL)](./IMDB-Movie-Analysis-SQL/)
Exploratory analysis of 80,000+ IMDB movie records using SQL to uncover genre trends, director performance, rating distributions, and the relationship between runtime and audience scores.

**Tools:** SQL · MySQL · Python · Matplotlib  
**Techniques:** Aggregation · Window Functions · CTEs · Exploratory Analysis

---

#### [🏦 Sales Data Warehouse & ETL Pipeline (SQL)](./Sales-Data-Warehouse-SQL/)
Designs and implements a star-schema data warehouse for retail sales transactions, including dimension/fact tables and an incremental refresh ETL pipeline — a core pattern in production data engineering.

**Tools:** SQL · MySQL / PostgreSQL  
**Techniques:** Star Schema · ETL · Stored Procedures · Incremental Refresh · Indexing

---

### 📊 Data Visualization & Dashboards

---

#### [🛒 Walmart Sales Dashboard (R Shiny)](./Walmart-Sales-RShiny/)
Interactive R Shiny dashboard for exploring 420,000+ weekly Walmart sales records across 45 stores and 99 departments. Features dynamic filtering, holiday impact analysis, and trend visualization.

**Tools:** R · R Shiny · ggplot2 · plotly · dplyr  
**Techniques:** Interactive Dashboard · Time-Series Visualization · Retail Analytics

---

### 🌫️ Data Integration & APIs

---

#### [🌫️ Weather & Air Quality Data Integration (NYC)](./Weather-Air-Quality-NYC/)
Integrates PM2.5 air quality data (EPA), weather observations (NOAA), and wind direction (Open-Meteo) for Albany and Syracuse, NY into a normalized MySQL database. Reveals the relationship between meteorological conditions and pollution levels through polar plots and correlation analysis.

**Tools:** Python · Pandas · Requests · SQLAlchemy · MySQL · Matplotlib  
**Techniques:** REST API Integration · ETL · Relational Database Design · Environmental Data Analysis

---

## 🛠️ Skills

| Category | Skills |
|----------|--------|
| **Languages** | Python · R · SQL |
| **ML & Modeling** | Scikit-learn · XGBoost · PyTorch · Regression · Classification · Clustering |
| **NLP & LLMs** | NLTK · HuggingFace Transformers · Gensim · AWS Bedrock · LangChain · FAISS |
| **Data & Databases** | Pandas · NumPy · MySQL · PostgreSQL · ETL · Data Warehousing |
| **Visualization** | Matplotlib · Seaborn · Plotly · Folium · R Shiny |
| **Tools** | Jupyter Notebook · Git · GitHub · Google Colab · AWS |

---

## 📬 Contact

- 💼 [LinkedIn]([https://www.linkedin.com/in/abyjoejose](https://www.linkedin.com/in/aby-joe-jose-88959021b/))
- 📧 [Email](abyjoejose00@gmail.com)

---

*Each project folder contains a dedicated README with problem statement, methodology, results, and instructions to run.*
