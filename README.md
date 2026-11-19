# Samsung Phone Query & Review System

An end-to-end intelligent assistant for querying Samsung smartphones, exploring GSMArena technical specifications, and generating multi-agent product reviews with zero hallucinations.

Built for the **Samsung Phone Query and Review System** assessment.

---

## 🌟 Key Capabilities

1. **Automated Web Scraping (GSMArena):**
   * Robust crawler built with `requests` and `BeautifulSoup4` extracting 10–15 Samsung models.
   * Polite request pacing and HTTP 429 backoff handling.
   * Regex extraction normalizing unstructured specs (battery mAh, display inches, RAM GB, launch prices).

2. **Relational Data Modeling (SQLAlchemy ORM):**
   * Multi-database architecture: Zero-configuration local **SQLite** (`phones.db`) for instant development, and containerized **PostgreSQL 16** via Docker Compose for production.
   * 3 relational tables: `phones`, `specifications` (lossless long-format EAV), and `prices`.
   * Fully idempotent ingestion (`app/ingest.py`) with upsert support.

3. **Conversational RAG Chatbot (Groq LLM / Localhost):**
   * Grounded question-answering with **zero hallucinations**.
   * Hybrid retrieval: Exact entity resolution (`find_mentioned`), dynamic catalogue leaderboards (best battery, highest RAM, screen sizes), and TF-IDF fallback search.
   * Cites exact phone source cards below every response.

4. **Multi-Agent Product Review Pipeline:**
   * Autonomous two-agent collaboration:
     * **`SpecRetrievalAgent` (Data Researcher):** Queries the relational database for verified specs and computes comparative catalogue rankings.
     * **`ReviewWriterAgent` (Tech Journalist):** Synthesizes an 8-section structured markdown review (Overview, Display, Performance, Camera, Battery & Charging, Pros, Cons, and Verdict).

5. **Production-Ready FastAPI Service & Web UI:**
   * Clean RESTful endpoints (`/chat`, `/review`, `/phones`, `/health`, `/api/verify-key`, `/api/models`).
   * Interactive web front-end running on `http://localhost:8000/`.
   * **In-browser API Key Management:** Enter your Groq API key directly in the UI, auto-detect enabled models with one click, or toggle to **Localhost (Ollama)** for 100% offline local inference.

---

## 🏛️ System Architecture

```
[1. Web Scraper]
   GSMArena.com ──(Search & Detail Crawl)──▶ Polite Delays ──▶ Parsed Specs (JSON)
                                                                     │
[2. Database Persistence]                                            ▼
   phones.db (SQLite / PostgreSQL) ◀──(Idempotent Upsert)── app/ingest.py
      ├── Table: phones (id, name, battery_mah, display_inches, ram_gb, announced)
      ├── Table: specifications (phone_id, category, key, value)
      └── Table: prices (phone_id, amount, currency, raw)
                                                                     │
[3. Intelligence & Agents]                                           ▼
   ┌────────────────────────────────────────┴────────────────────────────────────────┐
   ▼                                                                                 ▼
[RAG Engine (/chat)]                                             [Multi-Agent Pipeline (/review)]
User Query ──▶ Exact Entity Match (S23 vs S24)                   Phone Selection ──▶ SpecRetrievalAgent
     ├──▶ Pre-calculated Catalogue Rankings                                                 │ (Fetches Specs +
     └──▶ Strict Grounded Prompt Synthesis (Groq)                                           ▼  Catalogue Rank)
               │ (Answers ONLY from DB context)                                  SpecReport (Data Sheet)
               ▼                                                                            │
     Factual Answer + Source Citations                                                      ▼
                                                                                 ReviewWriterAgent (Groq)
                                                                                            │
                                                                                            ▼
                                                                                 Structured 8-Section Review
   └────────────────────────────────────────┬────────────────────────────────────────┘
                                            │
[4. Serving & Web UI]                       ▼
   FastAPI Endpoints ──▶ Web Browser (http://localhost:8000/)
   (Interactive Chatbot, Multi-Agent Reviewer, Device Catalog, In-Browser API Key Manager)
```

---

## 📁 Project Structure

```
samsung-phone-assistant/
├── app/
│   ├── agents.py          # Step 3: SpecRetrievalAgent & ReviewWriterAgent pipeline
│   ├── chatbot.py         # Step 2: Grounded RAG conversational chatbot
│   ├── config.py          # Central configuration & database URL
│   ├── db.py              # SQLAlchemy engine, session factory, table initialization
│   ├── ingest.py          # Idempotent database loader (JSON -> DB upserts)
│   ├── llm.py             # Groq LLM wrapper with auto-model discovery & Localhost support
│   ├── main.py            # Step 4: FastAPI REST API & static file routing
│   ├── models.py          # Relational schema (phones, specifications, prices)
│   ├── parsing.py         # Regex parsing for battery, display size, RAM, and price
│   ├── rag.py             # Entity matching, catalogue rankings, and TF-IDF retrieval
│   └── static/            # Step 4: Web UI (HTML5, modern CSS3, Vanilla ES6 JavaScript)
│       ├── index.html     # Interactive frontend layout & modals
│       ├── style.css      # Dark Samsung-tech responsive theme
│       └── app.js         # API Key manager, chat engine, agent runner, catalog viewer
├── data/
│   └── sample_phones.json # 12-phone verified specification dataset
├── scraper/
│   └── gsmarena.py        # Step 1: BeautifulSoup web scraper targeting GSMArena
├── tests/
│   ├── test_project.py    # Unit & integration tests (parsing, ingest, RAG, agents)
│   └── test_api_key.py    # Tests for API key resolution, verification, and endpoints
├── docker-compose.yml     # PostgreSQL 16 container definition
├── requirements.txt       # Python dependencies
├── .env.example           # Environment template
└── README.md              # Project documentation
```

---

## 🚀 Quickstart Guide

### 1. Clone & Set Up Virtual Environment

```bash
git clone https://github.com/adibgafur/samsung-phone-assistant.git
cd samsung-phone-assistant

# Create virtual environment
python -m venv .venv
source .venv/bin/activate        # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configure Environment (Optional)

```bash
cp .env.example .env
```
*(Note: You can also leave `.env` blank and configure your Groq API key or Localhost provider directly inside the web front-end!)*

### 3. Initialize Database & Ingest Phone Data

The project supports both **SQLite** (default zero-config) and **PostgreSQL**:

```bash
# Option A: Zero-config SQLite (default - instant setup)
python -m app.ingest data/sample_phones.json

# Option B: Production PostgreSQL (via Docker)
docker compose up -d
DATABASE_URL="postgresql+psycopg2://app:app@localhost:5432/phones" python -m app.ingest data/sample_phones.json
```

*(Optional: To run the live GSMArena scraper: `python -m scraper.gsmarena --out data/scraped_phones.json`)*

### 4. Start the Application

```bash
uvicorn app.main:app --reload
```

* **Web UI (Frontend):** [http://localhost:8000/](http://localhost:8000/)
* **Interactive API Docs (Swagger):** [http://localhost:8000/docs](http://localhost:8000/docs)
* **Health Check:** [http://localhost:8000/health](http://localhost:8000/health)

### 5. Run Automated Tests

```bash
pytest tests/test_api_key.py -v
pytest tests/test_project.py -k "not test_api" -v
```

---

## 🌐 Web UI & Front-End Features

* **In-Browser API Key Manager:** Click `⚙️ API Key / Host` in the navbar.
  * Enter your Groq API key (`gsk_...`) or choose **Localhost (Ollama / LM Studio)**.
  * **Auto-Detect Models:** Click `🔄 Detect Key Models` to dynamically fetch active models enabled on your specific account (e.g. `llama-3.1-8b-instant`, `openai/gpt-oss-20b`, `qwen/qwen3.8-27b`).
  * Keys are saved in browser `localStorage` and sent with requests via `X-API-Key` headers.
* **Conversational Chatbot:** Features one-click quick prompt chips, conversation memory, and citations of retrieved phones.
* **Multi-Agent Review System:** Select any device and watch the two-agent pipeline (`SpecRetrievalAgent` $\rightarrow$ `ReviewWriterAgent`) compose structured product reviews with copy and export features.
* **Phone Catalog & Spec Inspector:** Filter, search, and view complete category-by-category GSMArena specification sheets.

---

## 📡 REST API Reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Web UI (Interactive Frontend) |
| `GET` | `/health` | Health status and device count in DB |
| `GET` | `/phones` | Normalized summary of all phones |
| `GET` | `/phones/{name}` | Complete categorised specifications for a device |
| `GET` | `/api/key-status` | Connectivity status of client / server API keys |
| `POST` | `/api/models` | Dynamically list available models for an API key |
| `POST` | `/api/verify-key` | Test connection and model access (`{"api_key": "..."}`) |
| `POST` | `/chat` | RAG query (`{"question": "...", "session_id": "..."}`) |
| `POST` | `/review` | Multi-agent review (`{"phone": "Galaxy S24 Ultra"}`) |

---

## 🛠️ Design Decisions

* **RAG Without Embedding Fragility:** With consumer phone specs, vector embeddings frequently confuse model numbers (e.g. *S23* vs. *S23 Ultra*). Exact entity resolution combined with precalculated catalogue leaderboards guarantees accuracy.
* **Dual Database Support:** Running SQLite by default ensures any developer or recruiter can clone and launch the project in under 30 seconds, while SQLAlchemy ORM provides seamless enterprise scaling to PostgreSQL.
* **Idempotent Ingestion:** The scraper output can be re-ingested at any time without database corruption or duplicate records.
