# ⚡ EnergyRAG — Document Intelligence

A beautiful RAG chatbot for energy sector documents. Upload PDFs and ask precise,
cited questions — powered by Gemini 2.0 Flash and sentence-transformers.

## Local Setup

```bash
pip install -r requirements.txt
streamlit run app.py
```

## Deploy to Streamlit Cloud (Free)

1. Push this folder to a GitHub repo
2. Go to [share.streamlit.io](https://share.streamlit.io)
3. Connect your GitHub repo → select `app.py`
4. Click **Deploy**

No environment variables needed — API key is entered in the UI sidebar.

## Usage

1. Paste your Gemini API key (free at [aistudio.google.com](https://aistudio.google.com/app/apikey))
2. Upload one or more energy PDFs
3. Ask questions in the chat — answers cite document and page number
