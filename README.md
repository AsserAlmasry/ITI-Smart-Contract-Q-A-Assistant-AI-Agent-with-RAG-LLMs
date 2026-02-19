# 🔐 Smart Contract Q&A Assistant

> AI-powered legal & technical document analysis powered by **Gemini API**, **ChromaDB**, **LangChain**, **LangServe**, and **Gradio**.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Notebook Steps](#notebook-steps)
- [Setup & Installation](#setup--installation)
- [How to Run](#how-to-run)
- [Features](#features)
- [LangServe REST API](#langserve-rest-api)
- [Evaluation Pipeline](#evaluation-pipeline)
- [Known Issues & Fixes](#known-issues--fixes)
- [Limitations](#limitations)
- [License](#license)

---

## Overview

The **Smart Contract Q&A Assistant** is a production-grade Retrieval-Augmented Generation (RAG) system built to analyze legal and technical documents — including blockchain smart contracts, employment agreements, and any structured contract documents.

Users upload a document (PDF, DOCX, or TXT), and the system chunks, embeds, and indexes the content into a vector database. Questions are answered by retrieving the most relevant chunks and passing them to Google's Gemini model for grounded, citation-backed responses.

The system includes:
- A full **Gradio web UI** with document upload, Q&A chat, and source citations
- A **LangServe REST API** for programmatic access
- A **guardrails system** for input validation and output safety
- An **evaluation pipeline** with retrieval and answer quality metrics

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    USER INTERFACE LAYER                     │
│              Gradio Web UI  ·  LangServe REST API           │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    GUARDRAILS LAYER                         │
│         Input Validation  ·  Output Safety Checks           │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                      RAG PIPELINE                           │
│                                                             │
│  ┌─────────────┐    ┌──────────────┐    ┌────────────────┐  │
│  │  Document   │    │   Vector     │    │    Gemini      │  │
│  │  Ingester   │───▶│   Store     │───▶│  Generation    │  │
│  │  (Chunker)  │    │  (ChromaDB)  │    │    (LLM)       │  │
│  └─────────────┘    └──────────────┘    └────────────────┘  │
│                                                             │
│  Embeddings: sentence-transformers/all-MiniLM-L6-v2 (local) │
└─────────────────────────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                  EVALUATION LAYER                           │
│    Retrieval Metrics  ·  Answer Quality  ·  Report          │
└─────────────────────────────────────────────────────────────┘
```
<img width="3120" height="452" alt="download" src="https://github.com/user-attachments/assets/2be58e2c-7fa5-4e2c-98d1-9c0faa445f70" />

---

## Tech Stack

| Component | Technology | Purpose |
|---|---|---|
| **LLM** | Google Gemini 2.5 Flash Lite | Answer generation |
| **Embeddings** | `all-MiniLM-L6-v2` (HuggingFace) | Local semantic embeddings |
| **Vector Store** | ChromaDB | Chunk indexing & similarity search |
| **RAG Framework** | LangChain | Pipeline orchestration |
| **API Serving** | LangServe + FastAPI | REST API endpoints |
| **Web UI** | Gradio 4.x | Interactive chat interface |
| **Document Parsing** | python-docx, PyMuPDF, unstructured | PDF / DOCX / TXT ingestion |
| **Runtime** | Google Colab (Python 3.12) | Cloud notebook environment |

---

## Project Structure

```
Smart_Contract_QA_Assistant.ipynb
│
├── Step 1  — Package Installation & Runtime Setup
├── Step 2  — Imports, API Key, Global Config
├── Step 3  — Sample Smart Contract Generation
├── Step 4  — Core Components
│   ├── Component 1: DocumentIngester
│   ├── Component 2: VectorStoreManager
│   ├── Component 3: SmartContractGuardrails
│   └── Component 4: SmartContractRAGChain  ← RAG pipeline + LangServe Runnable
├── Step 5  — Component Initialization
├── Step 6  — Pipeline Testing (unit tests + guardrail tests)
├── Step 7  — Gradio UI + LangServe API Launch
└── Step 8  — Evaluation Pipeline & Report      ← NEW
```

---

## Notebook Steps

### Step 1 — Installation
Installs and pins all dependencies with Colab-compatible versions:
- `gradio==4.44.1` + `huggingface_hub==0.23.4` (compatibility fix)
- `langchain`, `langchain-google-genai`, `chromadb`
- `langserve==0.3.0` + `httpx==0.27.2` (compatibility fix)
- `sentence-transformers` for local embeddings
- Triggers automatic runtime restart after install

### Step 2 — Imports & Configuration
- Loads all libraries
- Reads `GEMINI_API_KEY` from Colab Secrets
- Defines global constants (chunk size, overlap, top-k, model name)

### Step 3 — Sample Contract
Writes a full synthetic smart contract (`sample_smart_contract.txt`) to `/content/` covering tokenomics, vesting, governance, staking, and sale terms for AlphaToken (ALPHA).

### Step 4 — Core Components

**`DocumentIngester`**
Handles ingestion of PDF, DOCX, and TXT files. Splits documents into overlapping chunks using LangChain's `RecursiveCharacterTextSplitter`. Tags each chunk with metadata (source filename, chunk ID).

**`VectorStoreManager`**
Wraps ChromaDB with methods to add, query, and clear document chunks. Uses `sentence-transformers/all-MiniLM-L6-v2` for local embeddings (no API quota consumed).

**`SmartContractGuardrails`**
Validates inputs (length, injection attempts, gibberish) and outputs (hallucination markers, unsafe content). Returns structured pass/fail with reason strings.

**`SmartContractRAGChain`**
The full RAG pipeline:
1. Input guardrails
2. ChromaDB retrieval (top-k chunks)
3. Context assembly
4. Gemini generation with structured prompt
5. Output guardrails
6. Citation formatting

Exposes `as_runnable()` for LangServe integration.

### Step 5 — Initialization
Instantiates all components and wires them together:
```python
ingester  = DocumentIngester()
vsm       = VectorStoreManager()
guardrails = SmartContractGuardrails()
rag_chain  = SmartContractRAGChain(vsm, guardrails)
```

### Step 6 — Testing
Runs 5 sample Q&A tests against the built-in contract and 5 guardrail tests (prompt injection, empty query, jailbreak attempts). Prints pass/fail for each.

### Step 7 — Gradio UI + LangServe
- Patches `gradio_client` schema parser bugs
- Starts LangServe on port 8001 in a background thread
- Launches the Gradio interface with public `gradio.live` share URL
- Enter key wired via JavaScript for Colab iframe compatibility

### Step 8 — Evaluation Pipeline
Runs a structured evaluation suite measuring retrieval precision, answer faithfulness, and response completeness. Outputs a full evaluation report with scores, findings, and recommendations.

---

## Setup & Installation

### Prerequisites
- Google Colab account (free tier works)
- Google Gemini API key ([get one here](https://aistudio.google.com/))

### Step-by-step

**1. Open the notebook in Google Colab**
```
File → Upload notebook → Select Smart_Contract_QA_Assistant.ipynb
```

**2. Add your API key to Colab Secrets**
```
🔑 Secrets panel (left sidebar) → Add new secret:
  Name:  GEMINI_API_KEY
  Value: your-api-key-here
```

**3. Run Step 1 (installation)**
The cell will install all packages and automatically restart the runtime. This is expected — do not panic.

**4. After restart, run Steps 2 through 8 in order**
Do NOT re-run Step 1 after the restart.

---

## How to Run

### Using the Gradio UI
1. Run all cells (Steps 2–8)
2. Click the `gradio.live` public URL printed at the end of Step 7
3. Upload a contract PDF/DOCX/TXT **or** click **Load Sample Contract**
4. Type a question and press **Enter** or click **Send ➤**

### Using the LangServe API
Once Step 7 is running, the REST API is available on port 8001:

```bash
# Single question (invoke)
curl -X POST http://localhost:8001/contract-qa/invoke \
  -H "Content-Type: application/json" \
  -d '{"input": "What is the total token supply?"}'

# Batch questions
curl -X POST http://localhost:8001/contract-qa/batch \
  -H "Content-Type: application/json" \
  -d '{"inputs": ["What is the vesting schedule?", "What are the staking rewards?"]}'

# Swagger docs
open http://localhost:8001/contract-qa/docs
```

---

## Features

| Feature | Description |
|---|---|
| 📄 Multi-format ingestion | PDF, DOCX, TXT support |
| 🔍 Semantic search | Local MiniLM embeddings, no API quota |
| 🤖 Grounded answers | Responses cite specific contract sections |
| 🚫 Input guardrails | Blocks injections, gibberish, empty queries |
| ✅ Output guardrails | Flags hallucination markers and unsafe content |
| 🔌 REST API | LangServe `/invoke`, `/batch`, `/stream` endpoints |
| 📊 Evaluation | Automated retrieval and answer quality metrics |
| 🗑️ DB isolation | Auto-clears DB on each new upload to prevent source mixing |
| ⌨️ Enter key support | JS workaround for Colab iframe keyboard limitation |

---

## LangServe REST API

The API is automatically started on port 8001 when Step 7 runs.

| Endpoint | Method | Description |
|---|---|---|
| `/contract-qa/invoke` | POST | Single question → single answer |
| `/contract-qa/batch` | POST | Multiple questions → multiple answers |
| `/contract-qa/stream` | POST | Streaming response (SSE) |
| `/contract-qa/docs` | GET | Interactive Swagger UI |
| `/contract-qa/playground` | GET | LangServe playground UI |

**Request format:**
```json
{
  "input": "What is the notice period for termination?"
}
```

**Response format:**
```json
{
  "output": ["The notice period is...", "📚 Sources:\n[1] contract.docx · chunk 3\n..."]
}
```

---

## Evaluation Pipeline (Step 8)

The evaluation step runs automatically after Step 7 and produces a structured report covering:

### Metrics

| Metric | Description | Target |
|---|---|---|
| **Retrieval Precision** | % of retrieved chunks relevant to the query | ≥ 0.70 |
| **Answer Faithfulness** | Answer grounded in retrieved context (no hallucination) | ≥ 0.80 |
| **Answer Completeness** | Key expected facts present in answer | ≥ 0.75 |
| **Guardrail Effectiveness** | Malicious inputs correctly blocked | 100% |
| **Latency (p50 / p95)** | Response time percentiles | < 5s / < 10s |

### Evaluation Questions
The pipeline tests against 10 predefined question/expected-answer pairs covering:
- Factual retrieval (token supply, vesting terms, jurisdictions)
- Multi-part answers (staking rewards, governance rights)
- Edge cases (missing information, template placeholders)
- Guardrail probes (injection, jailbreak, empty input)

### Output
A full evaluation report is printed in the notebook and optionally saved to `/content/evaluation_report.txt`.

---

## Known Issues & Fixes

| Issue | Cause | Fix Applied |
|---|---|---|
| `ImportError: HfFolder` | Colab's `huggingface_hub >= 0.24` removed `HfFolder` | Pinned to `0.23.4` + monkey-patch shim |
| numpy binary incompatibility | ABI mismatch in Colab | Force-reinstall `numpy==1.26.4` |
| `APIInfoParseError: Cannot parse schema True` | `gradio_client` bug in `_json_schema_to_python_type` | Monkey-patch wrapping both public and private functions |
| `ImportError: VerifyTypes` from httpx | `httpx >= 0.28` removed `VerifyTypes` used by LangServe | Pinned `httpx==0.27.2` |
| `[Errno 98] address already in use` | LangServe thread still running on cell re-run | `_langserve_started` guard prevents double-bind |
| Enter key not working in Colab | Colab iframe blocks Gradio's native submit event | JavaScript `keydown` listener clicks Send button programmatically |
| Mixed document sources in answers | Old chunks persisted in ChromaDB between uploads | `vsm.clear()` called at start of every upload and load operation |

---

## Limitations

- **Template placeholders:** Documents with unfilled placeholders (e.g., `[NAME]`, `[DATE]`) will return those placeholders in answers — this is correct behaviour, not a bug.
- **Context window:** Only the top-5 most similar chunks are passed to Gemini. Very long contracts may require increasing `top_k` or reducing chunk size.
- **Local embeddings:** `all-MiniLM-L6-v2` is fast and free but less powerful than Gemini's embedding API for domain-specific legal text.
- **No persistent storage:** ChromaDB runs in-memory in Colab. All indexed documents are lost when the runtime disconnects.
- **Colab timeouts:** Free Colab sessions disconnect after ~90 minutes of inactivity. Re-run Steps 2–8 after reconnecting.
- **LangServe port:** Port 8001 is only accessible from within the Colab runtime. To expose it publicly, use `ngrok` or Colab's port forwarding.
- **Single-document focus:** The system is optimised for one document at a time. Uploading multiple contracts simultaneously may reduce answer precision due to cross-document retrieval.

---

## License

This project is intended for educational and demonstration purposes.
All AI-generated answers should be verified by a qualified legal professional before acting on them.

---

*Built by ENG. Asser Almasry using Gemini API · ChromaDB · LangChain · LangServe · Gradio*
