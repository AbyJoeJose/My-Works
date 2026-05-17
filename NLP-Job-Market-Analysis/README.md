# 🔤 NLP Pipeline: Job Market Analysis

> An end-to-end Natural Language Processing pipeline applied to 50,000+ real job postings — from data collection and classical NLP through deep learning, LLMs, and a production-ready RAG system.

---

## 📌 Problem Statement

The job market generates vast amounts of unstructured text. This project builds a complete NLP pipeline to extract insights from job postings: classifying roles, identifying required skills, generating job descriptions, and building a semantic job search engine using RAG.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Source | ZipRecruiter (scraped via API + search results) |
| Size | 50,000+ job postings |
| Domain | Data Science, ML Engineering, Analytics, SQL roles |
| Format | JSON (job descriptions, company info, salary) |

---

## 🧠 Pipeline Modules

| Module | Technique | Tools |
|--------|-----------|-------|
| Data Collection & EDA | Regex, JSON parsing, date/URL extraction | Python, re, Pandas |
| Text Classification | CountVectorizer + Logistic Regression (16 scenarios) | Scikit-learn, NLTK |
| Tokenization | NLTK word tokenizer, BPE from scratch, Herdan's Law | HuggingFace Tokenizers |
| Word Vectors | TF-IDF, GloVe, Word2Vec, cosine similarity search | Gensim, Scikit-learn |
| Neural Networks | Feedforward net on TF-IDF features | PyTorch |
| Language Generation | Fine-tuned DistilGPT2, beam search, top-p, top-k | HuggingFace Transformers |
| NER / Skill Tagging | Fine-tuned DistilBERT (BIO tagging) | HuggingFace Transformers |
| Prompt Engineering | Structured JSON extraction with Mistral-7B | AWS Bedrock, boto3 |
| RAG System | FAISS vector store + Mistral-7B + LangChain | FAISS, LangChain, AWS Bedrock |

---

## 📈 Key Results

- **Text Classification:** ~83% macro F1 with best preprocessing scenario (lemmatization + bigrams)
- **NER:** DistilBERT correctly tags `B-SKILL` / `I-SKILL` entities with high precision on `O` class
- **RAG:** Semantic retrieval successfully surfaces relevant job postings for natural language queries
- **LLM Extraction:** Mistral-7B returns structured JSON skills with zero hallucinated programming languages

---

## 🛠️ Tools & Libraries

**Languages:** Python · **NLP:** NLTK, HuggingFace Transformers, Gensim, Tokenizers  
**ML/DL:** Scikit-learn, PyTorch · **LLMs:** AWS Bedrock (Mistral-7B), DistilGPT2, DistilBERT  
**RAG:** LangChain, FAISS, sentence-transformers · **Other:** BeautifulSoup, boto3, Pandas

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd "NLP-Job-Market-Analysis"
pip install -r requirements.txt
jupyter notebook NLP_Job_Market_Analysis.ipynb
```

> **Note:** Modules 6 (DistilGPT2 fine-tuning) and 7 (DistilBERT NER) were trained on Google Colab with GPU. Modules 8–9 require AWS Bedrock access.

---

## 🔑 Key Takeaways

- Lemmatization + bigrams consistently improves classification F1 over baseline
- GloVe semantic search outperforms TF-IDF cosine similarity for conceptual job queries
- Fine-tuned DistilBERT accurately identifies technical skills in job descriptions
- RAG architecture enables accurate, grounded answers to natural language job market queries

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
