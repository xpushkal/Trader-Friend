<div align="center">

```
██╗███╗   ██╗██╗   ██╗███████╗███████╗████████╗██╗ ██████╗
██║████╗  ██║██║   ██║██╔════╝██╔════╝╚══██╔══╝██║██╔═══██╗
██║██╔██╗ ██║██║   ██║█████╗  ███████╗   ██║   ██║██║   ██║
██║██║╚██╗██║╚██╗ ██╔╝██╔══╝  ╚════██║   ██║   ██║██║▄▄ ██║
██║██║ ╚████║ ╚████╔╝ ███████╗███████║   ██║   ██║╚██████╔╝
╚═╝╚═╝  ╚═══╝  ╚═══╝  ╚══════╝╚══════╝   ╚═╝   ╚═╝ ╚══▀▀═╝
```

**AI-powered investment intelligence for India's 14 crore+ retail investors**

[![ET AI Hackathon 2026](https://img.shields.io/badge/ET%20AI%20Hackathon-2026-1A3C6E?style=for-the-badge)](https://economictimes.com)
[![Problem Statement](https://img.shields.io/badge/PS%206-AI%20for%20Indian%20Investor-2563EB?style=for-the-badge)](https://economictimes.com)
[![Built with](https://img.shields.io/badge/Built%20with-LangGraph%20%7C%20FastAPI%20%7C%20React-00B4A6?style=for-the-badge)](#tech-stack)
[![License](https://img.shields.io/badge/License-MIT-22C55E?style=for-the-badge)](LICENSE)

</div>

---

## What is InvestIQ?

> India has 14 crore+ demat accounts. Most retail investors are flying blind — reacting to tips, missing filings, unable to read technicals, managing portfolios on gut feel. **InvestIQ is the intelligence layer that changes that.**

InvestIQ is a **multi-agent AI system** that continuously monitors Indian financial markets, detects technical chart patterns, classifies corporate signals, and delivers personalised, source-cited investment intelligence — in under 60 seconds from event to alert.

No guesswork. No generic news. Every signal is back-tested. Every claim is source-cited.

---

## Demo

```
Filing published on NSE  →  Ingestion Agent (60s poll)
  →  Signal Classifier (LLM: type · direction · urgency · confidence)
    →  Portfolio Context Agent (is this relevant to the user?)
      →  Alert Composition Agent (plain English + source citation + disclaimer)
        →  Delivery (ET App push / Email / WhatsApp Business)

Total latency target: < 90 seconds from filing to user alert
```

**Live demo flow (3-minute hackathon pitch):**
1. Trigger: Promoter bulk deal filed on NSE for a stock in the demo user's portfolio
2. System detects it in < 60 seconds, classifies it as `Bullish · High Confidence · Same-day urgency`
3. Chart Pattern Agent simultaneously detects a Bullish Flag breakout on the same stock
4. Market ChatGPT answers: *"Should I be paying attention to Reliance right now?"* — with 5-step reasoning, portfolio context, and cited sources

---

## Features

### 🔭 Opportunity Radar
Continuously monitors **BSE/NSE filings, bulk deals, insider trades, SEBI orders, and management commentary** — surfacing missed opportunities as daily alerts. Not a summariser. A signal-finder.

| Signal Type | Example | Urgency |
|---|---|---|
| Promoter bulk buy | 12.3L shares acquired at 2.1x avg. volume | Immediate |
| Insider trade | MD purchases shares 11 days before results | Same-day |
| SEBI order | Penalty lifted on previously flagged company | Weekly watch |
| Management commentary shift | Guidance upgraded in earnings call | Same-day |
| FII sector rotation | Rs 2,140 Cr inflow into financials today | Immediate |

### 📈 Chart Pattern Intelligence
Real-time technical pattern detection across the **full NSE universe (2,000+ stocks + F&O instruments)** with historical back-testing — not just what pattern appeared, but what happened next.

**Patterns detected (Phase 1):**
- Breakouts: Cup & Handle, Flag, Pennant, Triangle (ascending / descending / symmetrical)
- Reversals: Head & Shoulders, Double Top/Bottom, Morning/Evening Star
- Support/Resistance: Dynamic S/R zones, VWAP deviation bands, Historical pivot levels
- Momentum: RSI divergence, MACD crossover, Volume spike anomalies
- Candlestick: Engulfing, Doji, Hammer, Shooting Star (in trend-confluence only)

**Back-test output for every alert:**
```
Pattern:   Bullish Flag Breakout on NIFTY BANK (1H chart)
Hit rate:  74% (last 5 years, same pattern on same instrument)
Avg move:  +3.1% over 3–7 sessions when confirmed
Max DD:    -1.4% before resolution
Confidence: STRONG  |  Stop zone: 49,450  |  Measured target: 51,200
```

### 💬 Market ChatGPT — Next Gen
Portfolio-aware conversational AI with multi-step reasoning, session memory, and 100% source citation. Handles what the current ET Market ChatGPT can't:

```
User:  "If RBI cuts 25bps next week, how does my portfolio get hit?"
AI:    Step 1: Identifies rate-sensitive holdings in user's portfolio
       Step 2: Retrieves historical sensitivity of each sector to rate cuts
       Step 3: Pulls current NIM/credit data for banking stocks held
       Step 4: Models directional impact per holding
       Step 5: Synthesises portfolio-level delta with source citations
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                             │
│  NSE/BSE Filings  │  SEBI RSS  │  OHLCV Feed  │  FII/DII  │ ET │
└────────────────────────────┬────────────────────────────────────┘
                             │ 60-second poll
                    ┌────────▼────────┐
                    │ Ingestion Agent │  Normalises · Deduplicates
                    │   (Kafka bus)   │  Publishes to event stream
                    └────────┬────────┘
           ┌─────────────────┼──────────────────┐
           │                 │                  │
  ┌────────▼──────┐  ┌───────▼──────┐  ┌───────▼──────┐
  │ Signal        │  │ Pattern      │  │ Back-Test    │
  │ Classifier    │  │ Detection    │  │ Agent        │
  │ Agent (LLM)   │  │ Agent (TA)   │  │ (5yr OHLCV)  │
  └────────┬──────┘  └───────┬──────┘  └───────┬──────┘
           │                 │                  │
  ┌────────▼─────────────────▼──────────────────▼──────┐
  │              Signal Event Log  │  TimescaleDB       │
  │              OHLCV Store       │  Pinecone (RAG)    │
  └────────────────────┬───────────────────────────────┘
                       │
          ┌────────────▼──────────────┐
          │   Orchestration Layer      │  LangGraph · Retry logic
          │   (Task router + Audit)    │  Full decision audit trail
          └──────────┬────────────────┘
           ┌─────────┴──────────┐
  ┌────────▼──────┐    ┌────────▼───────────────┐
  │ Portfolio     │    │ Conversation Agent      │
  │ Context Agent │    │ (Market ChatGPT NG)     │
  │ User holdings │    │ Multi-turn · Source RAG │
  └────────┬──────┘    └────────────────────────┘
           │
  ┌────────▼──────────────────────────────────┐
  │        Alert Composition Agent            │
  │  LLM draft → source validation →          │
  │  SEBI disclaimer → hallucination check    │
  └────────┬──────────────────────────────────┘
           │
  ┌────────┴──────────────────────────────────┐
  │  ET App Push  │  Email Digest  │  WhatsApp │
  └───────────────────────────────────────────┘
```

**8 specialised agents. Every decision logged. Zero unsourced claims.**

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Agent orchestration | LangGraph | Multi-agent state machines, retry logic, audit trail |
| LLM backbone | GPT-4o / Claude Sonnet | Reasoning + structured JSON output generation |
| Data ingestion | Python + Kafka | Real-time, fault-tolerant event streaming |
| Technical analysis | TA-Lib + NumPy | Pattern detection + indicator computation |
| Time-series store | TimescaleDB | Efficient OHLCV queries + 5yr back-test scans |
| Vector store | Pinecone / Chroma | RAG over ET Markets corpus for ChatGPT agent |
| Backend API | FastAPI (Python) | Async, lightweight, auto-documented via OpenAPI |
| Frontend | React + TailwindCSS | Demo dashboard + alert UI |
| Delivery | Firebase + SMTP + WA Business API | Multi-channel alert routing |

---

## Project Structure

```
investiq/
├── agents/
│   ├── ingestion_agent.py          # Data source polling + normalisation
│   ├── signal_classifier.py        # LLM-based event classification
│   ├── pattern_detection.py        # TA-Lib pattern scanner
│   ├── backtest_agent.py           # Historical performance computation
│   ├── portfolio_context.py        # User holdings + watchlist injection
│   ├── alert_composition.py        # Source-cited alert generation
│   ├── conversation_agent.py       # Market ChatGPT multi-turn handler
│   └── delivery_agent.py           # Multi-channel alert routing
├── orchestration/
│   ├── graph.py                    # LangGraph agent coordination graph
│   ├── audit_trail.py              # Decision logging + replay
│   └── error_recovery.py           # Retry logic + fallback handling
├── data/
│   ├── ingestion/
│   │   ├── nse_feed.py             # NSE bulk deal + filing poller
│   │   ├── bse_feed.py             # BSE corporate filing scraper
│   │   ├── sebi_rss.py             # SEBI regulatory RSS parser
│   │   └── ohlcv_store.py          # TimescaleDB OHLCV manager
│   └── rag/
│       ├── corpus_builder.py       # ET Markets content indexer
│       └── retriever.py            # Pinecone query interface
├── api/
│   ├── main.py                     # FastAPI app entry point
│   ├── routes/
│   │   ├── alerts.py               # Alert query + history endpoints
│   │   ├── chat.py                 # Market ChatGPT conversation endpoint
│   │   └── portfolio.py            # User portfolio CRUD
│   └── schemas.py                  # Pydantic models
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── AlertFeed.jsx       # Real-time signal alert stream
│   │   │   ├── PatternCard.jsx     # Chart pattern + back-test display
│   │   │   ├── ChatInterface.jsx   # Market ChatGPT UI
│   │   │   └── PortfolioPanel.jsx  # User holdings context panel
│   │   └── App.jsx
│   └── package.json
├── tests/
│   ├── test_classifier.py
│   ├── test_pattern_detection.py
│   └── test_backtest.py
├── docker-compose.yml
├── .env.example
├── requirements.txt
└── README.md
```

---

## Quickstart

### Prerequisites

- Python 3.11+
- Node.js 18+
- Docker + Docker Compose
- API keys: OpenAI or Anthropic, Pinecone (optional for RAG)

### 1. Clone and configure

```bash
git clone https://github.com/your-team/investiq.git
cd investiq
cp .env.example .env
```

Edit `.env`:

```env
# LLM
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
LLM_PROVIDER=openai          # or: anthropic

# Data
NSE_API_KEY=                 # Optional — uses scraping fallback if not set
OHLCV_DB_URL=postgresql://localhost:5432/investiq

# Vector store (optional — needed for Market ChatGPT RAG)
PINECONE_API_KEY=
PINECONE_ENV=

# Delivery
FIREBASE_CREDENTIALS_PATH=./firebase-creds.json
SMTP_HOST=smtp.gmail.com
SMTP_USER=
SMTP_PASS=
WHATSAPP_API_TOKEN=          # Optional for hackathon demo
```

### 2. Start infrastructure

```bash
docker-compose up -d          # Starts Kafka, TimescaleDB, Redis
```

### 3. Install dependencies

```bash
# Backend
pip install -r requirements.txt

# Frontend
cd frontend && npm install
```

### 4. Seed historical data (for back-test demo)

```bash
python data/ingestion/ohlcv_store.py --seed --tickers RELIANCE,HDFCBANK,NIFTY50 --years 5
```

> For hackathon demo, 50 tickers × 5 years is sufficient. Full NSE universe seeding takes ~4 hours.

### 5. Run the system

```bash
# Terminal 1: Backend API
uvicorn api.main:app --reload --port 8000

# Terminal 2: Ingestion + agent pipeline
python orchestration/graph.py

# Terminal 3: Frontend demo
cd frontend && npm run dev
```

Open `http://localhost:5173` for the demo dashboard.

---

## Sample Outputs

### Opportunity Radar Alert
```json
{
  "ticker": "RELIANCE",
  "signal_type": "BULK_DEAL_PROMOTER",
  "direction": "BULLISH",
  "urgency": "SAME_DAY",
  "confidence": "HIGH",
  "summary": "Promoter entity acquired 12.3L shares via bulk deal at Rs 2,847 (2.1x avg daily volume).",
  "historical_context": "Last 4 similar promoter bulk buys on RIL preceded 8–14% gains over 90 days (3 of 4 instances).",
  "watch": "Q4 results due in 11 days. Bulk buy ahead of results historically signals confidence.",
  "sources": ["NSE Bulk Deal Data", "ET Markets Filing DB"],
  "disclaimer": "This is signal information, not investment advice. Past performance is not indicative of future results.",
  "timestamp": "2026-03-28T09:47:33Z",
  "latency_seconds": 73
}
```

### Pattern Alert
```json
{
  "instrument": "NIFTY BANK",
  "pattern": "BULLISH_FLAG_BREAKOUT",
  "timeframe": "1H",
  "confidence_band": "STRONG",
  "backtest": {
    "hit_rate": 0.74,
    "sample_size": 31,
    "avg_return_pct": 3.1,
    "median_sessions_to_target": 5,
    "max_drawdown_pct": 1.4
  },
  "levels": {
    "breakout": 49840,
    "target": 51200,
    "stop_loss": 49450
  },
  "confluence": "FII bought Rs 2,140 Cr in financials today — directional alignment.",
  "disclaimer": "Pattern detection only. Not investment advice."
}
```

---

## Impact Model

| Metric | Baseline | With InvestIQ |
|---|---|---|
| Filing → user alert | 6–48 hours | < 90 seconds |
| Chart pattern detection | Manual (hours) | Automated (15-min cycles) |
| Actionable signals/user/day | 0 (generic news) | 3–5 personalised |
| Source citation rate | 0% | 100% |
| Research time saved | — | ~4 hrs/week |

**Back-of-envelope value created:**
```
Target segment:    20L active equity investors on ET Markets
Avg capital:       Rs 3.5L per user
Signal uplift:     +1.5% annual return from better entry/exit timing
                   (conservative — 2–3 strong signals/month vs 0 currently)

Aggregate value:   20L × Rs 3.5L × 1.5% = Rs 1,050 Crore / year

ET monetisation:   Rs 999–2,999/year premium tier × 5L paying users
                 = Rs 500–1,500 Crore ARR
```

---

## Compliance & Guardrails

- **Not a SEBI-registered advisor.** Every output carries a mandatory disclaimer. The system surfaces signals — users make their own decisions.
- **No buy/sell recommendations.** Language is constrained at the composition layer. Pattern alerts show historical data, not predictions.
- **100% source-cited.** The Alert Composition Agent validates every claim against retrieved source data before output. Unsourced claims are blocked.
- **Hallucination detection.** LLM outputs are cross-checked against structured source data. Contradictions trigger a re-generation or human escalation flag.
- **User data privacy.** Portfolio data is encrypted at rest and in transit. Never used for model training without explicit opt-in.

---

## Hackathon Sprint Plan

| Phase | Time | Deliverable |
|---|---|---|
| Phase 0: Setup | 0–4h | Repo, environment, API keys, scaffolding |
| Phase 1: Data | 4–10h | NSE bulk deal feed + OHLCV data for 50 tickers |
| Phase 2: Signals | 10–16h | LLM classifier + alert composition end-to-end |
| Phase 3: Patterns | 16–28h | 3–5 patterns live with back-test stats |
| Phase 4: Chat | 28–36h | Market ChatGPT with 3 portfolio-aware demo queries |
| Phase 5: Polish | 36–44h | Frontend dashboard, alert UI, full demo flow |
| Phase 6: Submit | 44–48h | README, pitch video, architecture doc, impact model |

---

## Team

> Add your team members here

| Name | Role |
|---|---|
| — | Agent architecture + LangGraph orchestration |
| — | Data engineering (NSE/BSE ingestion + TimescaleDB) |
| — | Pattern detection + back-test engine |
| — | Frontend + demo dashboard |
| — | Product + pitch + impact model |

---

## Submission Checklist

- [ ] Public GitHub repository with commit history
- [ ] This README with setup instructions
- [ ] 3-minute pitch video (problem → solution → live demo)
- [ ] Architecture document (1–2 pages, see `/docs/architecture.md`)
- [ ] Impact model with quantified assumptions (see Impact Model section above)
- [ ] Working demo: Opportunity Radar + Chart Pattern alert end-to-end
- [ ] Working demo: Market ChatGPT with portfolio-aware query

---

## References

- [ET Markets](https://economictimes.indiatimes.com/markets) — Primary data context and inspiration
- [Avataar.ai](https://avataar.ai) — Hiring partner
- [NSE India](https://www.nseindia.com) — Market data source
- [BSE India](https://www.bseindia.com) — Filing data source
- [SEBI](https://www.sebi.gov.in) — Regulatory data source
- [LangGraph Docs](https://langchain-ai.github.io/langgraph/) — Agent orchestration framework

---

<div align="center">

**ET AI Hackathon 2026 · Problem Statement 6 · AI for the Indian Investor**

*Built with ❤️ for India's 14 crore retail investors*

</div>