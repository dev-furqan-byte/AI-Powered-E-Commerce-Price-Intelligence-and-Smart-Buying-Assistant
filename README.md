# Product Price & Quality Comparison System (PriceIntel AI)

A robust, verified, and test-driven multi-marketplace product comparison and price intelligence system built with **FastAPI**, **SQLite**, **RapidFuzz**, and **Groq LLaMA 3.3 70B**.

---

## 🌟 Core Architecture & Pipeline

```text
step6_complete_project/
├── app.py               # [1] REST API Controller & Static Web Dashboard Router
├── marketplaces.py      # [2] Multi-Store Asynchronous Connector (eBay, Best Buy, Amazon, Walmart)
├── engine.py            # [3] Algorithmic Intelligence (URL Parsing, Variant Matching, Quality Scoring, AI Explainer)
├── database.py          # [4] SQLite Persistence Layer (Time-Series Price History & Delta Tracking)
├── run.py               # [5] 1-Click Server Launcher with Environment Loading
├── test_system.py       # [6] Automated Test Suite (21 test suites validating all 10 criteria)
├── static/
│   └── index.html       # [7] Modern Glassmorphism Web Dashboard + Chart.js Line Graphs + History Table
├── .env.example         # [8] Documented Template for Optional Live API Keys
└── README.md            # Comprehensive Learning & Architectural Guide
```

---

## 🔄 End-to-End Data Flow (Request to Response)

1. **User Action**:
   The user enters a product title (e.g. *"iPhone 15"*, *"MacBook Air M2"*) or pastes a direct product URL (e.g. from Amazon, eBay, Best Buy, or Walmart) into the web dashboard.
2. **URL Resolution (`app.py` & `engine.py:ProductUrlParser`)**:
   If a URL is detected, the parser automatically strips query fragments, extracts the core product keywords, identifies the source marketplace, and extracts the SKU/ASIN/Item ID.
3. **Data Collection (`marketplaces.py`)**:
   Asynchronously queries the 4 target marketplaces (**eBay**, **Best Buy**, **Amazon**, and **Walmart**) in parallel:
   - If official API keys are configured (`EBAY_CLIENT_ID`, `BESTBUY_API_KEY`, `SERPAPI_API_KEY`), live endpoints are queried.
   - If credentials are absent or a store call fails, verified benchmark observations are seamlessly used.
   - Every listing is stamped with its collection timestamp (`timestamp`) and source tag (`live_api` vs `verified_benchmark`).
   - **Zero Price Fabrication Rule**: The system never invents fake prices or random deals.
4. **Variant & Hardware Spec Matching (`engine.py:check_variant_match`)**:
   - Extracts hardware specifications: Storage (128GB vs 256GB), RAM, and Model Series (Pro vs Pro Max vs Ultra).
   - Uses RapidFuzz token matching to compute confidence scores.
   - Flags variant discrepancies (e.g., *"Storage Mismatch: Requested 256GB, listing offers 128GB"*) to prevent false comparisons.
5. **5-Factor Multi-Criteria Quality Scoring (`engine.py:calculate_quality_score`)**:
   Produces an evidence-based **0 to 100 Quality Score** across 5 factors:
   - Customer Star Rating (40 pts)
   - Logarithmic Review Count Credibility (20 pts)
   - Seller Reputation & Trust % (20 pts)
   - Item Condition: Brand New (10 pts) vs Refurbished (6 pts) vs Used (3 pts)
   - Warranty Coverage: 2-Year (10 pts) vs 1-Year (8 pts) vs 90-Day (6 pts) vs None (0 pts)
6. **Deterministic Price Analysis (`engine.py:analyze_listings`)**:
   Calculates lowest price, highest price, absolute price spread ($\Delta$), percentage savings, average price, and identifies the **Best Value Pick** based on Quality Score per Dollar.
7. **Evidence-Based AI Explainer (`engine.py:generate_ai_explanation`)**:
   - Synthesizes factual trade-offs between the lowest price option and higher quality options using Groq LLaMA 3.3 70B (or strict deterministic rule fallback).
   - Strictly forbidden from hallucinating unavailable reviews or unlisted specifications.
8. **Time-Series Historical Preservation (`database.py:save_search_data`)**:
   - Never overwrites past records; saves every observation chronologically with a timestamp.
   - Automatically detects price changes (e.g. `🔥 Amazon Price Drop: Reduced by $50.00!`).
9. **Visual Rendering (`static/index.html`)**:
   Renders the Glassmorphism dashboard with KPI cards, multi-store deal cards, variant warning ribbons, an interactive Chart.js line graph, and a chronological observation history table.

---

## 📁 File-by-File Purpose & Specification

### 1. `marketplaces.py`
- **Exact Purpose**: Marketplace data retrieval module for eBay, Best Buy, Amazon, and Walmart.
- **Why Necessary**: Centralizes external API calls and fallback logic into a single non-blocking async module.
- **Input**: Search query string `query: str`.
- **Output**: Dictionary containing `canonical_name`, `brand`, `category`, `collection_timestamp`, and `listings` array with full pricing, condition, rating, warranty, and seller metadata.
- **Used by**: `app.py`.
- **Separation of Responsibility**: Contains zero UI code and zero database calls; strictly manages product offer collection.

### 2. `engine.py`
- **Exact Purpose**: Core intelligence layer containing URL parsing, hardware spec extraction, RapidFuzz variant matching, 0-100 quality scoring, deterministic price calculations, and the Groq/Rule AI explainer.
- **Why Necessary**: Prevents invalid comparisons (e.g., comparing a 128GB phone against a 256GB phone), determines true value mathematically, and explains trade-offs without hallucination.
- **Input**: Raw URLs or keywords, listing dictionaries, and price stats.
- **Output**: Clean query, match confidence, warning lists, 0-100 quality score breakdown, price statistics, and AI verdict.
- **Used by**: `app.py` and `test_system.py`.
- **Separation of Responsibility**: Pure functional logic without I/O or network side-effects.

### 3. `database.py`
- **Exact Purpose**: SQLite time-series storage and historical observation retrieval.
- **Why Necessary**: Preserves historical records over time so subsequent searches can track price drops and render trend lines.
- **Input**: Product query, canonical details, and processed listings.
- **Output**: SQLite `product_id`, price drop/increase alerts, Chart.js dataset structure, and chronological observation rows.
- **Used by**: `app.py` and `test_system.py`.
- **Separation of Responsibility**: Isolated data access layer with automatic schema migrations.

### 4. `app.py`
- **Exact Purpose**: FastAPI HTTP application controller.
- **Why Necessary**: Acts as the bridge connecting the web browser to the data connectors, intelligence engine, and database.
- **Endpoints**:
  - `GET /`: Serves the web dashboard.
  - `POST /api/compare`: Coordinates the full research and comparison pipeline.
  - `GET /api/history/{id}`: Returns historical chart and table data.
  - `GET /api/recent`: Returns recent searches.
  - `GET /api/status`: Returns system connectivity health.
  - `POST /api/config`: Dynamically updates live API keys in memory.
- **Used by**: Web browsers, HTTP clients, and `test_system.py`.

### 5. `static/index.html`
- **Exact Purpose**: Single-file modern web user interface.
- **Why Necessary**: Provides a professional, responsive Glassmorphism dashboard allowing users to search products, inspect verified listings, view AI advice, track historical charts, and configure API keys.
- **Used by**: End users in any modern browser.

### 6. `test_system.py`
- **Exact Purpose**: Pytest automated test suite.
- **Why Necessary**: Verifies that all 10 core acceptance criteria function correctly and prevents regressions.
- **Coverage**: 21 tests covering URL parsing, variant matching, quality scoring, price calculations, database persistence, delta alerts, and API routes.

### 7. `run.py`
- **Exact Purpose**: 1-click startup runner.
- **Why Necessary**: Configures Windows UTF-8 output, loads `.env` variables, and launches Uvicorn.

---

## 🚀 How to Run the Application

### 1. Launch the Server
In your terminal, run:

```bash
cd d:\My_work\E-commarce\step6_complete_project
python run.py
```

Open your browser at:
- 🌐 **Web Dashboard:** `http://127.0.0.1:8000`
- 📑 **Interactive API Docs (Swagger):** `http://127.0.0.1:8000/docs`

### 2. Run the Automated Tests
In your terminal, execute:

```bash
python -m pytest test_system.py -v
```

All 21 test suites will run and report status.

---

## 🔑 Optional Live API Credentials

The system works **100% out of the box** using verified benchmark catalog observations. To enable live API querying, configure keys via `.env` or in the web UI using the **"API Keys"** button:

| Service | Purpose | Where to Obtain (Free) |
| :--- | :--- | :--- |
| **Groq API Key** | LLaMA 3.3 70B AI Explanations | [console.groq.com](https://console.groq.com) |
| **eBay Client ID & Secret** | Official eBay Browse API (5,000 calls/day free) | [developer.ebay.com](https://developer.ebay.com) |
| **Best Buy Developer API** | Official Best Buy Developer Catalog | [developer.bestbuy.com](https://developer.bestbuy.com) |
| **SerpApi Key** | Real-time Amazon & Walmart organic listings | [serpapi.com](https://serpapi.com) |
