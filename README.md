# Beca 18 RAG Chatbot — Document Retrieval and Grounded Generation

## Purpose

This project builds an end-to-end **Retrieval-Augmented Generation (RAG)** pipeline that answers user questions about the official Beca 18 regulations (PRONABEC, Peru) by retrieving relevant fragments from the source PDF and passing them as context to Google's Gemini 2.5 Flash model.

**Source document:** Resolución Directoral Ejecutiva N.° 033-2026-MINEDU/VMGI-PRONABEC

The system never relies on the model's parametric knowledge and declines to answer when the information is not present in the document.

## Pipeline Summary

The pipeline extracts text from the regulation PDF page by page, applies light cleaning, counts tokens with `tiktoken`, and splits the text into 400-token chunks (60-token overlap) using LangChain's `RecursiveCharacterTextSplitter`. Each chunk is embedded with `gemini-embedding-001` (3072 dimensions, RETRIEVAL_DOCUMENT task type) and stored in a persistent ChromaDB collection using cosine distance. At query time, the user's question is embedded (RETRIEVAL_QUERY task type), the top-k nearest chunks are retrieved, and Gemini 2.5 Flash generates a grounded answer using only the retrieved context.

## Installation and Setup

**Python ≥ 3.10** is required.

It is recommended to use a virtual environment:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

**API Key Configuration:**

1. Get a free API key from [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
3. Edit `.env` and replace `your_key_here` with your actual key:
   ```
   GEMINI_API_KEY=AIza...your_actual_key...
   ```

> **Important:** Never commit the `.env` file to the repository.

## How to Run

1. Clone the repo and install dependencies
2. Download `beca18_reglamento.pdf` from [gob.pe](https://www.gob.pe/institucion/pronabec/normas-legales/7778068-033-2026-minedu-vmgi-pronabec) and place it in `data/`
3. Configure the `.env` file with your Gemini API key
4. Open `notebooks/beca18_rag_chatbot.ipynb` in Jupyter (or Google Colab)
5. Run all cells in order (Step 0 → Step 7)

The ChromaDB collection is persistent — on the first run it embeds all chunks; subsequent runs skip the embedding step automatically.

## How to Use the Chat Interface

After running all cells, Step 7 displays an interactive widget interface:

- **Text area:** Type your question about the Beca 18 regulations
- **k slider:** Control how many document fragments to retrieve (1–10; default 5)
- **"Preguntar" button:** Submit the question and generate a grounded answer
- **"Limpiar" button:** Clear the input and output
- **Accordion panel:** Expand to see the retrieved source fragments with page numbers and cosine distances

## Repository Structure

```
beca18-rag-chatbot/
├── data/
│   └── beca18_reglamento.pdf         # Source PDF (NOT committed)
├── notebooks/
│   └── beca18_rag_chatbot.ipynb      # Main RAG pipeline notebook
├── .env.example                       # API key template
├── .gitignore
├── requirements.txt
└── README.md
```

## Security Note

The `.env` file is listed in `.gitignore` and must **never** be committed. The `.env.example` template shows the required variable name only. Committing an API key deducts 2 points from the assignment grade.
