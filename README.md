# ⚡ EnergyRAG — Energy Document Intelligence

> Ask natural language questions against your energy sector PDFs and get **precise, cited answers** — powered by Google Gemini 2.5 Flash, local embeddings, and ChromaDB.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=flat-square&logo=python)
![Streamlit](https://img.shields.io/badge/Streamlit-1.35%2B-red?style=flat-square&logo=streamlit)
![Gemini](https://img.shields.io/badge/Gemini-2.5%20Flash-orange?style=flat-square&logo=google)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

---

## What Is This?

EnergyRAG is a **Retrieval-Augmented Generation (RAG)** system built for energy industry professionals. Upload any PDF — operational reports, maintenance manuals, HSE filings, capital project documents — and chat with it in plain English.

Every answer is:
- Grounded **strictly** in the uploaded documents (no hallucination)
- Cited with the **document name and page number**
- Generated at **temperature 0** for maximum factual reliability

---

## Demo

| Sidebar open | Sidebar collapsed |
|---|---|
| Upload PDFs, configure settings | Full-width centered chat |

**Example questions you can ask:**
- *"What was the total hydrocarbon production in Q3?"*
- *"What safety incidents are described and on which pages?"*
- *"Summarise the capital expenditure plans for next quarter."*
- *"What environmental compliance measures are mentioned?"*

---

## Architecture

```
PDF Upload → PDFProcessor → TextChunker → GeminiEmbedder → VectorStore (ChromaDB)
                                                                      ↓
User Question → GeminiEmbedder → VectorStore.search() → RAGChain → Gemini 2.5 Flash → Answer
```

| Component | Technology | Notes |
|---|---|---|
| PDF Extraction | PyMuPDF (`fitz`) | Page-by-page, cleans hyphenation & whitespace |
| Chunking | tiktoken `cl100k_base` | 500 token window, 50 token overlap |
| Embeddings | `sentence-transformers` `all-MiniLM-L6-v2` | Fully local, 384-dim, no API needed |
| Vector Store | ChromaDB (cosine similarity) | Persistent on disk, survives restarts |
| LLM | Google Gemini 2.5 Flash | 1 API call per question |
| Frontend | Streamlit | Dark themed chatbot UI |

---

## Quickstart

### 1. Clone

```bash
git clone https://github.com/YOUR_USERNAME/rag-energy-intelligence.git
cd rag-energy-intelligence
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

> First run downloads the embedding model (~90 MB, cached locally after that).

### 3. Get a free Gemini API key

Go to [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey) — it's free, no credit card needed.

### 4. Run

```bash
streamlit run app.py
```

Paste your API key in the sidebar, upload a PDF, and start asking questions.

---

## Deploy Free on Streamlit Cloud

1. Fork this repo on GitHub
2. Go to [share.streamlit.io](https://share.streamlit.io) → **New app**
3. Select your repo → set main file to `app.py`
4. Click **Deploy**

No environment variables needed — the API key is entered in the UI sidebar.

> ⚠️ **Note:** Streamlit Cloud has an ephemeral filesystem. The `chroma_db/` folder is wiped on each redeployment, so you'll need to re-upload your PDFs after deploying a new version.

---

## Project Structure

```
rag-energy-intelligence/
│
├── app.py                          # Streamlit app — full RAG pipeline + UI
├── RAG_Energy_Intelligence.ipynb   # Jupyter notebook — same pipeline, for analysis
├── requirements.txt                # All dependencies
├── .streamlit/
│   └── config.toml                 # Dark theme config
│
├── NorthSea_Energy_Q3_Report.pdf   # Sample test document
├── HLD_EnergyRAG.pdf               # High Level Design document
└── LLD_EnergyRAG.pdf               # Low Level Design document
```

> **Do not commit** `chroma_db/` — add it to `.gitignore`. It is auto-created on first run.

---

## How It Works

### Ingestion (run once per document)

1. **Extract** — PyMuPDF reads each page, fixes hyphenation, strips noise
2. **Chunk** — tiktoken splits text into 500-token windows with 50-token overlap
3. **Embed** — `all-MiniLM-L6-v2` converts each chunk to a 384-dim vector locally
4. **Store** — ChromaDB persists vectors + metadata (doc name, page number) to disk

### Query (every question)

1. **Embed** — user question is converted to a 384-dim vector
2. **Search** — ChromaDB finds the top-K most similar chunks (cosine similarity)
3. **Prompt** — chunks are formatted as labelled excerpts with doc/page/score
4. **Generate** — Gemini 2.5 Flash produces a grounded answer at temperature 0

### Why local embeddings?

The Google Gemini free tier does not include embedding model access. Using `sentence-transformers` locally means **zero cost and zero API quota** for embeddings — only the LLM generation step (1 call per question) uses the Gemini API.

---

## Configuration

All tunable parameters are at the top of `app.py`:

```python
EMBEDDING_MODEL = "all-MiniLM-L6-v2"   # local embedding model
LLM_MODEL       = "gemini-2.5-flash"   # Gemini generation model
CHROMA_PATH     = "./chroma_db"         # vector store location
CHUNK_SIZE      = 500                   # tokens per chunk
CHUNK_OVERLAP   = 50                    # overlap between chunks
TOP_K           = 5                     # chunks retrieved per query (UI: 3–10)
```

---

## Gemini Free Tier Limits

| Model | Requests/min | Requests/day |
|---|---|---|
| gemini-2.5-flash | 15 RPM | 100 RPD |


---

## Requirements

```
streamlit>=1.35.0
google-genai>=0.5.0
sentence-transformers>=2.7.0
chromadb>=0.5.0
pymupdf>=1.24.0
tiktoken>=0.7.0
```

Python 3.10 or higher recommended.

---

## Notebook

`RAG_Energy_Intelligence.ipynb` contains the same pipeline as the Streamlit app — same classes, same config, same prompt — wrapped in a Jupyter display layer for development and analysis.

```python
system = EnergyRAGSystem()
system.upload_pdf("NorthSea_Energy_Q3_Report.pdf")
system.ask("What maintenance was performed on Alpha Platform?")
```

---

## Built With

- [Streamlit](https://streamlit.io) — Python web UI framework
- [Google Gemini](https://aistudio.google.com) — LLM generation
- [sentence-transformers](https://www.sbert.net) — local text embeddings
- [ChromaDB](https://www.trychroma.com) — vector database
- [PyMuPDF](https://pymupdf.readthedocs.io) — PDF processing
- [tiktoken](https://github.com/openai/tiktoken) — token counting

---
