# 🧠 NLP Pipeline: Job Market Analysis

> An end-to-end Natural Language Processing pipeline applied to real-world job posting data scraped from **ZipRecruiter** — progressively building from regex-based data collection through classical NLP, deep learning, large language models, and a full RAG system. Built for IA653 (Natural Language Processing) at Clarkson University.

---

## 📌 Project Overview

This project treats the job market as an NLP corpus, applying 9 progressive modules to extract insights, classify postings, generate text, tag skills, and answer natural language queries about jobs — all using the same underlying dataset of real job postings.

---

## 🗂️ Pipeline Overview

| # | Module | Core Technique |
|---|--------|---------------|
| 1 | Data Collection & EDA | Regex, JSON parsing, date extraction, acronym analysis |
| 2 | Text Classification Baseline | CountVectorizer, Logistic Regression, 16 preprocessing scenarios |
| 3 | Tokenization & Vocabulary | NLTK, BPE tokenizer, Herdan's Law |
| 4 | Word Vectors | TF-IDF, GloVe, Word2Vec, cosine similarity |
| 5 | Neural Networks | PyTorch feedforward network, TF-IDF + dense layers |
| 6 | Language Generation | DistilGPT2 fine-tuning, beam search, top-p/top-k |
| 7 | Named Entity Recognition | DistilBERT fine-tuning, BIO tagging, skill extraction |
| 8 | Prompt Engineering & LLMs | AWS Bedrock, Mistral-7B, structured JSON extraction |
| 9 | RAG System | FAISS vector store, LangChain, Mistral-7B on Bedrock |

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | ZipRecruiter job postings (scraped) |
| Format | JSON files per job, organized in `data/search_results/` and `data/jobs/` |
| Key fields | `rawCanonicalZipJobPageUrl`, `form_location`, `form_search`, `page_number`, `jobDetails.Description`, `jobDetails.Salary`, `jobDetails.CompanyDetails.Description` |
| Unique identifier | `jid` — extracted from URL via regex (`jid=(\w+)`) |
| Note | Same job (`jid`) can appear in multiple search queries — not a unique key per row |
| RAG subset | 500 jobs sampled (random_state=42) |
| LLM module | 10 jobs from `homework_08_jobs.json` |

---

## 🔍 Module 1 — Data Collection & EDA

**Parsing job postings with regex and JSON:**

```python
# URL extraction from search result JSON files
urls = re.findall(r'"rawCanonicalZipJobPageUrl"\s*:\s*"([^"]+)"', text)

# Job ID extraction from URL
match = re.search(r"jid=(\w+)", url)
```

**Key findings:**

- Jobs were loaded from nested folder structure: `data/search_results/{textfolder}/{file}.json`
- `jid` is **not unique** — same job can appear across multiple search queries (grouped by `jid`, aggregated search terms into a list)
- **Top acronyms** found in job descriptions (via `r'\b[A-Z]{3,4}\b'`): **SQL, AWS, ETL, NLP, GCP, LLM, PTO, EEO, API, RAG**
- **Date extraction** — 4 regex patterns covering: `January 31, 2024` · `Jan 31, 2024` · `10/10/2025` · `2025-08-09`
- **Top 5 URLs** extracted from job descriptions using `r'\bhttps://(?:www\.)?\w+\.(?:com|gov|org)\b'`

---

## 📝 Module 2 — Text Classification Baseline

**Task:** Classify job postings by job category (multi-class)

**Setup:**
- Filtered to jobs with **exactly one search label** (single-label classification)
- HTML stripped with `BeautifulSoup`
- **Stratified 80/20 train/test split** (`random_state=37`) — required due to uneven class distribution
- Baseline: `CountVectorizer(max_features=5000)` + `LogisticRegression(max_iter=1000)`

**16 preprocessing scenarios** evaluated via **10-fold cross-validation**, varying:
- Lemmatization: True / False
- Stop word removal: True / False
- N-gram range: unigrams (1,1) / bigrams (1,2)
- Binary counts: True / False

**Best scenario:** Lemmatization + bigrams, evaluated by macro F1

**Baseline result:** ~**83% F1** (macro) on test data

**Key insight:** Labels are **NOT evenly distributed** — stratified splits are essential. The confusion matrix reveals which job categories are hardest to distinguish.

---

## 🔤 Module 3 — Tokenization & Vocabulary Analysis

**Tokenization pipeline on first job description:**
- `word_tokenize` → total tokens, unique tokens, top 15 frequency words
- Cleaning: regex filter → lowercase → stopword removal

**Stemming vs Lemmatization comparison (first 20 tokens):**

| Approach | Behavior |
|----------|---------|
| `PorterStemmer` | Strips affixes aggressively (e.g. "running" → "run") |
| `WordNetLemmatizer` | Returns dictionary root form (context-aware) |

**Herdan's Law — vocabulary growth:**
- Sampled vocabulary size every 100 words across the full corpus
- Vocabulary grows rapidly at first, then decelerates — new words become rarer as corpus grows
- Confirmed the sub-linear growth predicted by Herdan's Law

**BPE Tokenizer (trained from scratch):**
- `vocab_size=30,000` · special token: `<|endoftext|>`
- Pre-tokenizer: `ByteLevel(add_prefix_space=False)`
- Post-processor: `ByteLevel(trim_offsets=False)`
- Trained with `tokenizers.trainers.BpeTrainer` on all job descriptions via iterator

**Regex vs BPE vocabulary growth:**
- BPE vocabulary plateaus near its maximum (~27k) — controlled and bounded
- Regex vocabulary grows without bound — unlimited but noisier

---

## 📐 Module 4 — Word Vectors: TF-IDF, GloVe & Word2Vec

**Text cleaning pipeline** (applied before all vectorization):
- BeautifulSoup HTML stripping → lowercase → regex tokenization → remove punctuation → lemmatize → remove stopwords

**TF-IDF:**
- `TfidfVectorizer()` on all cleaned descriptions → matrix shape: `(num_jobs × num_terms)`
- Top 10 TF-IDF terms extracted per job
- Pairwise cosine similarity on upper triangle only (removes self-comparison)
- Similarity = 1.0 indicates **duplicate job descriptions** under different JIDs

**GloVe (glove-wiki-gigaword-50):**
- 50-dimensional pretrained embeddings loaded via `gensim.downloader`
- Job representations: **average GloVe vector** across all tokens in description
- Semantic search: query → avg GloVe → cosine similarity → top 5 most relevant jobs
- Example: `data ↔ information` similarity >> `data ↔ banana` similarity

**Word2Vec (trained on job corpus):**

| Model | `sg` | Window | Min Count | Vector Size |
|-------|------|--------|-----------|------------|
| CBOW | 0 | 5 | 2 | 100 |
| Skip-gram | 1 | 5 | 2 | 100 |

**Key insight:** GloVe semantic search outperforms TF-IDF for conceptual queries (e.g. "machine learning engineering healthcare big data cloud") because it captures semantic similarity rather than exact term overlap.

---

## 🔥 Module 5 — Neural Networks (PyTorch)

**Architecture — `DenseTFIDFModel`:**
- Input: TF-IDF sparse matrix converted to dense tensors (`max_features=10,000`, bigrams)
- Layers: `Linear(10000, 256)` → `ReLU` → `Dropout(0.3)` → `Linear(256, 256)` → `ReLU` → `Linear(256, num_classes)`
- Configurable: `n_hidden` hidden layers, `dropout` rate

**Data split:**
- 80% train → 80% train / 20% validation (inner split)
- 20% held-out test
- Stratified on class label throughout

**Training:**
- Loss: `CrossEntropyLoss`
- Optimizer: `Adam(lr=1e-3)`
- Epochs: **20**
- Batch size: **32**
- Metrics tracked per epoch: train loss, validation loss, train F1 (macro), validation F1 (macro)

**Learning curves plotted:** Loss and F1 over 20 epochs for both train and validation — confirms model converges without excessive overfitting.

**Key insight:** PyTorch feedforward net with TF-IDF + bigrams improves over the sklearn Logistic Regression baseline by capturing more expressive non-linear combinations of features.

---

## 💬 Module 6 — Language Generation (DistilGPT2 Fine-tuning)

> ⚠️ Run on **Google Colab with GPU** (fp16 training)

**Dataset:** 1,000 job descriptions sampled (random_state=42), 80/10/10 split via HuggingFace `DatasetDict`

**Tokenization:** DistilGPT2 tokenizer with `eos_token` as `pad_token`, block size = **128 tokens**

**Pre-trained generation (3 strategies compared):**

| Strategy | Parameters | Behavior |
|----------|-----------|----------|
| Beam search | `num_beams=5`, `no_repeat_ngram_size=1`, `do_sample=True` | Most coherent, avoids repetition |
| Top-p sampling | `top_p=0.85` | Nucleus sampling, natural variation |
| Top-k sampling | `top_k=25` | Restricts to top 25 tokens per step |

**Prompt:** `"Finish this job posting: Come join Acme Industries and our team of Data Scientists"`

**Fine-tuning configuration:**

| Parameter | Value |
|-----------|-------|
| Epochs | 3 |
| Train batch size | 4 |
| Eval batch size | 8 |
| Learning rate | 2e-5 |
| Weight decay | 0.01 |
| Mixed precision | fp16 |
| Eval strategy | per epoch |

**Key insight:** Top-k sampling produced the best log-probability scores. Fine-tuned model generates more job-domain-appropriate text compared to the base DistilGPT2.

---

## 🏷️ Module 7 — Named Entity Recognition: Skill Tagging (DistilBERT)

> ⚠️ Run on **Google Colab with GPU**

**Task:** Token classification using **BIO scheme** to tag skills in job descriptions

**Labels:** `O` · `B-SKILL` · `I-SKILL`

**Dataset split:** 80 / 10 / 10 (train / validation / test), `seed=42`

**Subword alignment:** `tokenize_and_align_labels()` maps BIO labels to subword tokens — continuation subwords and padding positions get label `-100` (ignored in loss)

**Model:** `distilbert/distilbert-base-uncased` for token classification

**Training configuration:**

| Parameter | Value |
|-----------|-------|
| Epochs | 5 |
| Learning rate | 1e-5 |
| Train batch size | 8 |
| Eval batch size | 8 |
| Weight decay | 0.01 |
| Eval strategy | per epoch |

**Evaluation metric:** `seqeval` — precision, recall, F1, accuracy at the entity level

**Inference pipeline:**
```python
ner_pipeline = pipeline('token-classification',
                         model='./ner-distilbert',
                         aggregation_strategy='simple')
```

**Key insight:** Model performs best on `O` (non-skill) tokens. `I-SKILL` (continuation of skill span) is harder — subword tokenization fragments multi-word skills, making boundary detection challenging. Output: skill words with entity group and confidence score.

---

## 🤖 Module 8 — Prompt Engineering & LLM Skill Extraction

**Model:** Mistral-7B (`mistral.mistral-7b-instruct-v0:2`) via **AWS Bedrock**

**Prompt template (Mistral instruction format):**
```
Given job description {job}
[INST] Please extract the job skills and provide in the JSON format with the following structure
{sample_output} DO NOT INCLUDE ANY OTHER TEXT OTHER THAN THE JSON RESPONSE.
If there are no skills just return an empty JSON object.[/INST]
```

**Expected JSON output structure:**
```json
{
  "programming_languages": ["Python", "R", "SQL"],
  "technical_skills": ["machine learning", "data science"],
  "other": ["communication skills"]
}
```

**Cost estimation** (tokenizer: `mistralai/Mistral-7B-Instruct-v0.2`):

| Token type | Rate |
|-----------|------|
| Input | $0.00015 / 1k tokens |
| Output | $0.00020 / 1k tokens |

**API call configuration:**
- `max_tokens=512` · `temperature=0.5`
- JSON extracted from response using `re.compile(r'\{.*\}', re.DOTALL | re.MULTILINE)`

**Hallucination check:** Verified that all extracted `programming_languages` actually appear (case-insensitive) in the source job description text.

**Key insight:** Mistral-7B successfully extracted structured JSON skills with **no hallucinated programming languages** across all 10 tested job descriptions. The strict JSON-only prompt instruction is critical to avoid prose responses.

---

## 🔎 Module 9 — Retrieval-Augmented Generation (RAG)

**Full pipeline:**

```
Job descriptions → clean → chunk (LangChain) → embed (sentence-transformers)
       → FAISS index → retriever (k=10) → prompt → Mistral-7B → answer
```

**Embedding model:** `sentence-transformers/all-MiniLM-L6-v2`

**Token analysis (before chunking):**
- Mean tokens per job: varies (plotted char count vs token count scatter)
- Chunk size: **2,000 characters** with **0 overlap** — avoids tokenizer truncation for most postings
- Average ratio: ~1 char ≈ less than 1 token for job description prose

**Document chunking:**
```python
splitter = RecursiveCharacterTextSplitter(chunk_size=2000, chunk_overlap=0)
```

Each chunk stored as `Document` with metadata: `jid`, `job_title`, `salary_range`

**FAISS index:** Built from 500-job corpus, saved locally and reloaded with `allow_dangerous_deserialization=True`

**LLM:** `mistral.mistral-7b-instruct-v0:2` via `ChatBedrock` — `temperature=0.2`, `max_tokens=700`

**RAG chain (LangChain):**
```python
rag_chain = ({'input': RunnablePassthrough(), 'context': retriever | format_docs}
              | prompt | llm | StrOutputParser())
```

**System prompt:** "Answer ONLY using the provided context. If the answer is not in the context, say 'I don't know.'"

**Sample queries tested:**

| Query | Behavior |
|-------|---------|
| Remote work frequency | Accurately retrieved and summarized postings mentioning remote work |
| Companies hiring for NLP | Retrieved NLP-tagged postings with company names |
| Highest Data Scientist salary | Retrieved relevant postings; salary range less reliable due to unstructured salary field |

**Key insight:** FAISS + Mistral-7B accurately retrieves semantically relevant postings for conceptual queries. Salary extraction is less reliable because salary data is inconsistently formatted across job postings.

---

## 📈 Results Summary

| Module | Key Result |
|--------|-----------|
| Data Collection | Parsed job postings via regex; top acronyms: SQL, AWS, ETL, NLP, GCP, LLM |
| Text Classification | Logistic Regression baseline ~**83% F1** (macro); best scenario: lemmatization + bigrams |
| Tokenization | Confirmed Herdan's Law; BPE vocab plateaus near **27k** — more controlled than regex |
| Word Vectors | GloVe semantic search outperforms TF-IDF for conceptual queries |
| Neural Networks | PyTorch feedforward net improves over baseline with dense TF-IDF + bigrams |
| Language Generation | Fine-tuned DistilGPT2; **top-k sampling** produced best log-probability |
| NER / Skill Tagging | DistilBERT fine-tuned for BIO skill extraction; best on `O`, struggles on `I-SKILL` |
| Prompt Engineering | Mistral-7B extracted structured JSON skills with **no hallucinated languages** |
| RAG System | FAISS + Mistral-7B accurately retrieves relevant postings; salary range less reliable |

---

## 🛠️ Tools & Libraries

| Category | Tools |
|----------|-------|
| **Language** | Python 3 |
| **Data** | `pandas`, `numpy`, `json`, `re`, `pathlib`, `collections.Counter` |
| **NLP Classical** | `nltk` (tokenize, stopwords, stem, lemmatize), `scikit-learn` (CountVectorizer, TfidfVectorizer, LogisticRegression, Pipeline, GridSearchCV) |
| **Web Scraping** | `beautifulsoup4` (HTML stripping) |
| **Tokenizers** | `tokenizers` (HuggingFace BPE), `transformers` (AutoTokenizer) |
| **Word Vectors** | `gensim` (GloVe via downloader, Word2Vec) |
| **Deep Learning** | `PyTorch` (nn, optim, DataLoader, TensorDataset) |
| **Transformers** | `transformers` (DistilGPT2, DistilBERT, AutoModelForCausalLM, AutoModelForTokenClassification, Trainer, TrainingArguments) |
| **Evaluation** | `evaluate` (seqeval for NER), `sklearn.metrics` (F1, confusion matrix) |
| **LLM / Cloud** | `boto3`, AWS Bedrock (Mistral-7B), `langchain`, `langchain_aws`, `langchain_community` |
| **Vector Search** | `FAISS` (via LangChain), `sentence-transformers/all-MiniLM-L6-v2` |
| **Visualization** | `matplotlib`, `seaborn` |
| **Environment** | Jupyter Notebook (local) + Google Colab (GPU modules 6 & 7) |

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd NLP-Job-Market-Analysis
pip install -r requirements.txt
jupyter notebook
```

**Run modules in order:**
```
Module_1_Data_Collection_EDA.ipynb
Module_2_Text_Classification.ipynb
Module_3_Tokenization.ipynb
Module_4_Word_Vectors.ipynb
Module_5_Neural_Networks.ipynb
Module_6_Language_Generation.ipynb        # Run on Google Colab (GPU)
Module_7_NER_Skill_Tagging.ipynb          # Run on Google Colab (GPU)
Module_8_Prompt_Engineering.ipynb         # Requires AWS Bedrock credentials
Module_9_RAG_System.ipynb                 # Requires AWS Bedrock credentials
```

**AWS Bedrock setup (modules 8 & 9):**
```bash
aws configure  # Set region to us-east-1
# Ensure Mistral-7B model access is enabled in your Bedrock console
```

**Required packages:**
```
pandas numpy matplotlib seaborn scikit-learn
nltk gensim beautifulsoup4 tokenizers transformers
datasets evaluate torch langchain langchain-aws
langchain-community faiss-cpu sentence-transformers boto3
```

---

## 📁 File Structure

```
NLP-Job-Market-Analysis/
├── data/
│   ├── search_results/          # Raw ZipRecruiter search JSONs
│   └── jobs/                    # Individual job posting JSONs
├── job_assignments_sampled.json # Job file assignment per student
├── job_jid.json                 # Merged job dataset with JIDs
├── homework_08_jobs.json        # 10 jobs for LLM module
├── ner_dataset/                 # HuggingFace NER dataset (BIO-tagged)
├── faiss_index/                 # Saved FAISS vector index
├── distilgpt2-jobs/             # Fine-tuned DistilGPT2 checkpoint
├── ner-distilbert/              # Fine-tuned DistilBERT NER checkpoint
├── Module_1_Data_Collection_EDA.ipynb
├── Module_2_Text_Classification.ipynb
├── Module_3_Tokenization.ipynb
├── Module_4_Word_Vectors.ipynb
├── Module_5_Neural_Networks.ipynb
├── Module_6_Language_Generation.ipynb
├── Module_7_NER_Skill_Tagging.ipynb
├── Module_8_Prompt_Engineering.ipynb
├── Module_9_RAG_System.ipynb
└── README.md
```

---

## 🔑 Key Takeaways

- **Regex is foundational** — before any ML, structured extraction of job IDs, acronyms, URLs, and dates from raw JSON gives you the schema to work with
- **`jid` is not a unique key** — same job appears across multiple search queries; always aggregate before modeling to avoid data leakage
- **Top technical skills in postings** — SQL, AWS, ETL, NLP, GCP, LLM, RAG signal the tech stack employers actually value, directly from the raw text
- **Stratified splits are non-negotiable** — job category distribution is heavily skewed; ignoring stratification would bias all classification metrics
- **Herdan's Law holds on job corpora** — vocabulary growth decelerates as expected; BPE tokenization is more controlled (plateaus at ~27k) vs. regex (unbounded)
- **GloVe beats TF-IDF for conceptual search** — exact word match (TF-IDF) fails for semantically related but lexically different queries; dense embeddings handle this naturally
- **PyTorch over sklearn for extensibility** — the feedforward net uses the same TF-IDF features but a non-linear transformation, which improves performance while keeping interpretability
- **BIO label alignment is tricky** — subword tokenization (DistilBERT) splits words into pieces; continuation subwords must get label -100, not a copied label, to avoid training on artifacts
- **Prompt engineering discipline** — instructing Mistral-7B to return ONLY JSON (with a concrete structure example) eliminates prose preamble and makes response parsing reliable
- **RAG grounds LLMs in evidence** — without retrieval, Mistral-7B would hallucinate job details; with FAISS context (k=10), answers are grounded in actual postings, and the system correctly says "I don't know" when the answer isn't in context

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
