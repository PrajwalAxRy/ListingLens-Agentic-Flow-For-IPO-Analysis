# Implementation Plan: ListingLens IPO-Analyst-AGI (V2)

**Version:** 2.0  
**Target Audience:** Junior-to-Mid Level Engineers  
**Environment:** Windows 10/11 with Docker/WSL2  
**Timeline:** No hard deadline (quality-focused development)  
**Status:** Production-Ready Architecture with Real-Time Monitoring

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Technology Stack Recommendations](#2-technology-stack-recommendations)
3. [System Requirements](#3-system-requirements)
4. [High-Level Architecture](#4-high-level-architecture)
5. [Development Environment Setup](#5-development-environment-setup)
6. [Project Structure](#6-project-structure)
7. [Development Roadmap](#7-development-roadmap)

---

## 1. Executive Summary

### 1.1 System Purpose

ListingLens IPO-Analyst-AGI is an event-driven, multi-agent system that mimics elite financial research analysts to predict IPO listing performance. The system:

- Ingests RHP/DRHP and Anchor Investor List PDFs
- Performs deep analysis using specialized AI agents
- Monitors real-time market data (GMP, subscription) during 3-day IPO windows
- Conducts qualitative due diligence (news, sentiment, promoter/anchor reputation)
- Generates predictions: **"Successful"** (>10% gain), **"Neutral"** (0-10%), or **"Unsuccessful"** (<0%)
- Continuously re-synthesizes predictions as new information arrives
- Provides auditable sources for all qualitative claims

### 1.2 Core Goals

- **Real-Time Intelligence:** Monitor market dynamics via dual-scheduler (3h quantitative + 8h qualitative loops)
- **Event-Driven Updates:** Automatically update predictions when new data arrives via Redis messaging
- **Heuristic Balance:** Avoid "hype traps" by weighing institutional demand against fundamental risks
- **Auditability:** Cite all sources for human verification
- **Production-Grade:** Scalable, testable, and maintainable architecture

### 1.3 Success Criteria

- ✅ Accurate extraction of financial data from RHP PDFs (>95% accuracy vs. manual review)
- ✅ Real-time GMP/subscription tracking with <5 min latency
- ✅ Qualitative research with minimum 5 cited sources per report
- ✅ Prediction updates within 2 minutes of new data arrival
- ✅ System uptime >99% during 3-day IPO windows
- ✅ Complete audit trail for all predictions

---

## 2. Technology Stack Recommendations

### 2.1 Core Framework

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| **Orchestration** | CrewAI | Built-in agent management, task orchestration, and tool integration. Simplifies multi-agent workflows. |
| **LLM Provider** | Perplexity API (market research) + Hugging Face Models (other tasks) | Perplexity for deep search; HF for cost-effective text generation (e.g., Llama-3, Mistral). |
| **Message Queue** | Redis 7.x | Pub/sub for event-driven agent communication; fast in-memory storage for current state. |
| **Time-Series DB** | PostgreSQL 15 with TimescaleDB extension | Stores subscription metrics, state history, and velocity calculations. |
| **Vector DB** | ChromaDB | Persistent mode for RAG; supports metadata filtering for multi-document queries. |
| **Scheduler** | APScheduler | Python-native cron-like scheduler for dual-loop triggers. |
| **CLI Framework** | Typer | Modern, type-safe CLI with auto-generated help. |
| **Logging** | Loguru | Structured JSON logging with rotation and filtering. |
| **PDF Processing** | pdfplumber + tabula-py | Text extraction + table parsing for RHP/Anchor docs. |
| **Web Scraping** | Playwright | Headless browser for GMP/subscription data (handles JavaScript rendering). |
| **Testing** | pytest + pytest-asyncio | Unit and integration testing with async support. |
| **Containerization** | Docker + Docker Compose | Isolated services for Redis, PostgreSQL, and application. |

### 2.2 Financial Data API Recommendation

**Recommendation:** screener.in API (India-focused) + Manual CSV Fallback

#### Why screener.in?

- Free tier available for Indian equities
- Provides P/E, P/B, RoE, revenue growth for peer companies
- JSON API for programmatic access
- Fallback: Manual CSV upload for peer financial data

#### Alternative Options:

- **Alpha Vantage** (Free tier: 5 API calls/min, 500/day) - Global coverage but India data limited
- **Yahoo Finance** via yfinance library (Free, scraping-based) - Unreliable for Indian unlisted companies
- **NSE India API** (Free, official) - Best for listed peers

**Decision:** Use screener.in API with NSE API fallback for listed peers.

### 2.3 Development Tools (Windows-Specific)

| Tool | Purpose | Installation |
|------|---------|--------------|
| **WSL2** | Linux environment on Windows | `wsl --install` (PowerShell as Admin) |
| **Docker Desktop** | Container runtime | Download from docker.com, enable WSL2 integration |
| **Windows Terminal** | Modern CLI interface | Microsoft Store or GitHub releases |
| **VS Code** | IDE with remote WSL extension | Download from code.visualstudio.com |
| **Python 3.11** | Stable, modern Python | Download from python.org or via WSL |
| **Git for Windows** | Version control | Download from git-scm.com |

---

## 3. System Requirements

### 3.1 Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| **FR-1** | Ingest RHP/DRHP PDF and extract financial data (Revenue, PAT, OFS%, Pledged Shares) | CRITICAL |
| **FR-2** | Ingest Anchor Investor List PDF and extract investor names, allocations | CRITICAL |
| **FR-3** | Scrape live GMP and subscription data every 3 hours during market hours | CRITICAL |
| **FR-4** | Perform qualitative research (news, sentiment) every 8 hours (24/7) | CRITICAL |
| **FR-5** | Calculate velocity metrics (QIB growth rate, GMP momentum) | CRITICAL |
| **FR-6** | Research promoter and anchor reputations with cited sources | CRITICAL |
| **FR-7** | Generate heuristic-based prediction (Successful/Neutral/Unsuccessful) | CRITICAL |
| **FR-8** | Automatically re-synthesize prediction when new data arrives | CRITICAL |
| **FR-9** | Persist all state changes with timestamps to PostgreSQL | CRITICAL |
| **FR-10** | Provide CLI commands for initialization, monitoring, and reporting | HIGH |
| **FR-11** | Export reports in JSON, Markdown, and HTML formats | HIGH |
| **FR-12** | Review final thesis for logical consistency | MEDIUM |

### 3.2 Non-Functional Requirements

| ID | Requirement | Acceptance Criteria |
|----|-------------|---------------------|
| **NFR-1** | Latency: Prediction updates within 2 min of new data | Measured via timestamp diffs in PostgreSQL |
| **NFR-2** | Accuracy: Data extraction >95% accurate vs. manual review | Validated against ground truth documents |
| **NFR-3** | Availability: System uptime >99% during IPO windows | Monitor scheduler health checks |
| **NFR-4** | Auditability: All qualitative claims have ≥2 cited sources | Enforced in agent output schemas |
| **NFR-5** | Scalability: Support concurrent analysis of 3 IPOs | Test with parallel Docker containers |
| **NFR-6** | Maintainability: Modular code with <20% code duplication | SonarQube or manual review |
| **NFR-7** | Testability: Core modules have ≥70% test coverage | pytest-cov coverage reports |
| **NFR-8** | Security: API keys stored in environment variables | Never hardcode secrets |

---

## 4. High-Level Architecture

### 4.1 System Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE                          │
│                  (CLI Tool: ipo-analyst)                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR AGENT                           │
│  • State Management (PostgreSQL)                                │
│  • Event Listener (Redis Pub/Sub)                               │
│  • Heuristic Synthesis Prompt                                   │
└───┬─────────────────────────────────────────────────┬───────────┘
    │                                                 │
    │ ◄─── COLD ANALYSIS (Run Once) ────►            │
    │                                                 │
    ▼                                                 ▼
┌──────────────────────┐                  ┌──────────────────────┐
│  Data Extractor      │                  │   Risk Auditor       │
│  + Validator         │                  │   (Red Flags)        │
└──────┬───────────────┘                  └──────────┬───────────┘
       │                                              │
       ▼                                              ▼
┌──────────────────────┐                  ┌──────────────────────┐
│  Fundamental         │                  │   RAG Query Tool     │
│  Analyst (Valuation) │                  │   (Vector DB)        │
└──────────────────────┘                  └──────────────────────┘
                             
    │ ◄─── HOT LOOPS (Event-Driven) ────►            │
    │                                                 │
    ▼                                                 ▼
┌──────────────────────┐                  ┌──────────────────────┐
│ 3-HOUR SCHEDULER     │                  │ 8-HOUR SCHEDULER     │
│ (Market Hours Only)  │                  │ (24/7 Operation)     │
└──────┬───────────────┘                  └──────────┬───────────┘
       │                                              │
       ▼                                              ▼
┌──────────────────────┐                  ┌──────────────────────┐
│ Quantitative Pulse   │                  │ Qualitative Research │
│ Agent                │                  │ Agent                │
│ • Scrape GMP         │                  │ • Hype Analysis      │
│ • Subscription Data  │                  │ • Diligence Check    │
│ • Velocity Analysis  │                  │ • Source Citations   │
└──────┬───────────────┘                  └──────────┬───────────┘
       │                                              │
       └──────────────┬──────────────┬────────────────┘
                      ▼              ▼
           ┌─────────────────────────────────┐
           │     REDIS MESSAGE BUS           │
           │  Channels:                      │
           │  • ipo_quant_updates            │
           │  • ipo_qual_updates             │
           └─────────────┬───────────────────┘
                         │
                         ▼
           ┌─────────────────────────────────┐
           │  ORCHESTRATOR RE-SYNTHESIS      │
           │  • Load Current State           │
           │  • Compare New vs Previous      │
           │  • Generate Updated Thesis      │
           │  • Persist to PostgreSQL        │
           └─────────────┬───────────────────┘
                         │
                         ▼
           ┌─────────────────────────────────┐
           │     REVIEW AGENT                │
           │  (Sanity Check Logic)           │
           └─────────────┬───────────────────┘
                         │
                         ▼
           ┌─────────────────────────────────┐
           │  OUTPUT LAYER                   │
           │  • JSON Reports                 │
           │  • Markdown Summaries           │
           │  • HTML Dashboards              │
           └─────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    DATA PERSISTENCE                             │
├─────────────────┬─────────────────────┬─────────────────────────┤
│  PostgreSQL     │  ChromaDB           │  Redis                  │
│  • Metrics      │  • Vector Embeddings│  • Message Queue        │
│  • State Logs   │  • RHP/Anchor Docs  │  • Current State Cache  │
└─────────────────┴─────────────────────┴─────────────────────────┘
```

### 4.2 Data Flow

#### Cold Analysis (Initialization):

1. User uploads RHP + Anchor PDFs via CLI
2. Ingestion layer extracts text and tables
3. ChromaDB generates embeddings and stores chunks
4. Data Extractor Agent queries RAG and produces fact sheets
5. Validator verifies numerical accuracy
6. Fundamental Analyst performs valuation analysis
7. Risk Auditor identifies red flags
8. Orchestrator generates baseline prediction

#### Hot Loop 1 (Quantitative - 3h):

1. APScheduler triggers at 9:15 AM, 12:15 PM, 3:15 PM (market hours)
2. Quantitative Pulse Agent scrapes GMP + subscription data
3. Agent fetches previous metrics from PostgreSQL
4. Calculates velocity (% change, momentum)
5. Generates bullish/bearish signal
6. Publishes report to Redis channel `ipo_quant_updates`
7. Orchestrator receives event, re-synthesizes prediction
8. Persists new state to PostgreSQL

#### Hot Loop 2 (Qualitative - 8h):

1. APScheduler triggers every 8 hours (24/7)
2. Qualitative Research Agent runs two tasks:
   - **Hype Task:** Searches news/analyst reports via Perplexity API
   - **Diligence Task:** Researches promoters + anchors with citations
3. Agent publishes reports to Redis channel `ipo_qual_updates`
4. Orchestrator receives event, re-synthesizes prediction
5. Persists new state to PostgreSQL

#### Re-Synthesis:

1. Orchestrator loads current state from PostgreSQL
2. Compares new report with previous data
3. Invokes LLM with comprehensive prompt (all reports)
4. Parses structured prediction (Successful/Neutral/Unsuccessful)
5. Review Agent validates logical consistency
6. Final report generated with sources

---

## 5. Development Environment Setup

### 5.1 Windows Prerequisites

#### Step 1: Enable WSL2 (Windows Subsystem for Linux)

Open PowerShell as Administrator and run:

```powershell
# Enable WSL
wsl --install

# This installs Ubuntu by default
# Restart your computer when prompted
```

After restart, verify installation:

```powershell
wsl --list --verbose
# Should show Ubuntu running (version 2)
```

#### Step 2: Install Docker Desktop

1. Download Docker Desktop from: https://www.docker.com/products/docker-desktop/
2. Run installer
3. **IMPORTANT:** During installation, select "Use WSL 2 instead of Hyper-V"
4. After installation, open Docker Desktop
5. Go to Settings → Resources → WSL Integration
6. Enable integration with your Ubuntu distribution
7. Apply & Restart

Verify Docker works:

```powershell
docker --version
# Should show: Docker version 24.x.x or higher

docker run hello-world
# Should download and run a test container
```

#### Step 3: Install Windows Terminal (Optional but Recommended)

```powershell
# Via Microsoft Store or download from GitHub
# Search "Windows Terminal" in Microsoft Store
```

#### Step 4: Install VS Code with Remote WSL Extension

1. Download VS Code: https://code.visualstudio.com/
2. Install "Remote - WSL" extension
3. Open VS Code, press `Ctrl+Shift+P`, type "WSL: Connect to WSL"

### 5.2 WSL Ubuntu Setup

Open WSL terminal (type `wsl` in PowerShell or use Windows Terminal):

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Python 3.11
sudo apt install -y python3.11 python3.11-venv python3-pip

# Verify Python
python3.11 --version
# Should show: Python 3.11.x

# Install Git
sudo apt install -y git

# Install system dependencies for PDF processing
sudo apt install -y libpoppler-cpp-dev poppler-utils

# Install Java (required for tabula-py)
sudo apt install -y default-jre
```

### 5.3 Docker Compose Services Setup

Create `docker-compose.yml` in project root:

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: ipo_redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  postgres:
    image: timescale/timescaledb:latest-pg15
    container_name: ipo_postgres
    environment:
      POSTGRES_DB: ipo_agi
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: changeme123
      PGDATA: /var/lib/postgresql/data/pgdata
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init_db.sql:/docker-entrypoint-initdb.d/init_db.sql
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d ipo_agi"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  redis_data:
  postgres_data:
```

Start services:

```bash
# In WSL terminal, navigate to project directory
cd /mnt/c/Users/YourUsername/Projects/ipo-analyst

# Start Docker services
docker-compose up -d

# Verify services are running
docker-compose ps
# Should show redis and postgres as "Up"

# Check logs
docker-compose logs -f
```

#### Common Docker Commands Reference:

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (fresh start)
docker-compose down -v

# Restart a specific service
docker-compose restart redis

# View logs for specific service
docker-compose logs -f postgres

# Execute command in running container
docker-compose exec postgres psql -U admin -d ipo_agi
```

### 5.4 Python Virtual Environment Setup

```bash
# In WSL, navigate to project directory
cd /mnt/c/Users/YourUsername/Projects/ipo-analyst

# Create virtual environment
python3.11 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Upgrade pip
pip install --upgrade pip

# Install core dependencies
pip install crewai crewai-tools
pip install redis psycopg2-binary chromadb
pip install pdfplumber tabula-py
pip install playwright
pip install apscheduler
pip install typer loguru
pip install python-dotenv pydantic
pip install pytest pytest-asyncio pytest-cov

# Install Playwright browsers (for web scraping)
playwright install chromium

# Verify installations
python -c "import crewai; print(crewai.__version__)"
python -c "import redis; print('Redis OK')"
```

Create `requirements.txt`:

```txt
# Core Framework
crewai==0.30.0
crewai-tools==0.2.0
langchain==0.1.20
langchain-community==0.0.38

# LLM APIs
openai==1.12.0
anthropic==0.18.0
huggingface-hub==0.20.0

# Database
redis==5.0.1
psycopg2-binary==2.9.9
chromadb==0.4.22

# Document Processing
pdfplumber==0.10.3
tabula-py==2.9.0
pypdf2==3.0.1

# Web Scraping
playwright==1.41.0
beautifulsoup4==4.12.3
lxml==5.1.0

# Scheduling
apscheduler==3.10.4

# CLI
typer==0.9.0
rich==13.7.0

# Logging
loguru==0.7.2

# Utilities
python-dotenv==1.0.0
pydantic==2.6.1
pyyaml==6.0.1

# Testing
pytest==8.0.0
pytest-asyncio==0.23.4
pytest-cov==4.1.0
pytest-mock==3.12.0

# Code Quality
black==24.1.1
ruff==0.2.0
```

### 5.5 Environment Variables Setup

Create `.env` file in project root:

```bash
# LLM API Keys
PERPLEXITY_API_KEY=your_perplexity_api_key_here
HUGGINGFACE_API_KEY=your_huggingface_token_here

# Database Configuration
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=ipo_agi
POSTGRES_USER=admin
POSTGRES_PASSWORD=changeme123

REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0

# ChromaDB
CHROMA_PERSIST_DIR=./data/vector_store

# Scraping Configuration
GMP_SCRAPER_USER_AGENT=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
SCRAPER_RATE_LIMIT_DELAY=2

# Scheduler Configuration
SCHEDULER_TIMEZONE=Asia/Kolkata
MARKET_HOURS_START=09:15
MARKET_HOURS_END=15:30

# System Configuration
LOG_LEVEL=INFO
LOG_FILE=./logs/ipo_analyst.log
MAX_RETRIES=3
RETRY_DELAY=5
```

Create `.gitignore`:

```gitignore
# Environment
.env
venv/
__pycache__/
*.pyc
*.pyo
*.pyd
.Python

# Data
data/
logs/
*.pdf
*.csv

# IDE
.vscode/
.idea/
*.swp
*.swo

# Testing
.pytest_cache/
.coverage
htmlcov/

# Build
dist/
build/
*.egg-info/
```

### 5.6 Verification Checklist

```bash
# ✅ WSL2 installed and running
wsl --list --verbose

# ✅ Docker services running
docker-compose ps

# ✅ Python virtual environment activated
which python  # Should point to venv/bin/python

# ✅ All dependencies installed
pip list | grep crewai
pip list | grep redis

# ✅ Environment variables loaded
python -c "from dotenv import load_dotenv; load_dotenv(); import os; print(os.getenv('POSTGRES_HOST'))"

# ✅ PostgreSQL accessible
docker-compose exec postgres psql -U admin -d ipo_agi -c "SELECT version();"

# ✅ Redis accessible
docker-compose exec redis redis-cli ping
# Should return: PONG
```

---

## 6. Project Structure

```
ipo-analyst/
├── README.md
├── requirements.txt
├── .env
├── .gitignore
├── docker-compose.yml
├── init_db.sql
├── pyproject.toml
│
├── src/
│   ├── __init__.py
│   │
│   ├── core/                          # Core system components
│   │   ├── __init__.py
│   │   ├── config.py                  # Configuration loader
│   │   ├── logger.py                  # Loguru setup
│   │   ├── state.py                   # State management dataclasses
│   │   └── database.py                # PostgreSQL/Redis connection managers
│   │
│   ├── ingestion/                     # Document ingestion layer
│   │   ├── __init__.py
│   │   ├── pdf_parser.py              # PDF text/table extraction
│   │   ├── chunker.py                 # Text chunking for RAG
│   │   └── embedder.py                # Vector embedding generation
│   │
│   ├── tools/                         # CrewAI tools
│   │   ├── __init__.py
│   │   ├── rag_query_tool.py          # Query ChromaDB
│   │   ├── gmp_scraper_tool.py        # Web scraping for GMP data
│   │   ├── market_research_tool.py    # Perplexity API wrapper
│   │   ├── peer_data_tool.py          # Screener.in API
│   │   └── velocity_calculator.py     # PostgreSQL time-series queries
│   │
│   ├── agents/                        # CrewAI agents (specialized roles)
│   │   ├── __init__.py
│   │   ├── data_extractor.py          # Extract facts from RHP
│   │   ├── validator.py               # Cross-check numerical data
│   │   ├── fundamental_analyst.py     # Valuation analysis
│   │   ├── risk_auditor.py            # Identify red flags
│   │   ├── quantitative_pulse.py      # 3-hour market metrics agent
│   │   ├── qualitative_research.py    # 8-hour news/diligence agent
│   │   └── review.py                  # Thesis sanity checker
│   │
│   ├── crews/                         # CrewAI crew definitions
│   │   ├── __init__.py
│   │   ├── cold_analysis_crew.py      # One-time static analysis
│   │   ├── quantitative_pulse_crew.py # 3-hour market metrics loop
│   │   └── qualitative_research_crew.py # 8-hour news/diligence loop
│   │
│   ├── orchestrator/                  # Central control
│   │   ├── __init__.py
│   │   ├── orchestrator.py            # Main state manager
│   │   ├── event_listener.py          # Redis pub/sub handler
│   │   └── synthesis.py               # Heuristic prediction logic
│   │
│   ├── scheduler/                     # Dual-loop scheduling
│   │   ├── __init__.py
│   │   ├── scheduler.py               # APScheduler configuration
│   │   └── triggers.py                # Market hours logic
│   │
│   ├── cli/                           # Typer CLI interface
│   │   ├── __init__.py
│   │   ├── main.py                    # CLI entry point
│   │   ├── commands_init.py           # Initialization commands
│   │   ├── commands_analyze.py        # Analysis commands
│   │   └── commands_report.py         # Report generation
│   │
│   └── utils/                         # Utility functions
│       ├── __init__.py
│       ├── validation.py              # Schema validators
│       ├── formatting.py              # Report formatters (JSON/MD/HTML)
│       └── error_handling.py          # Retry logic, circuit breakers
│
├── tests/                             # Test suite
│   ├── __init__.py
│   ├── conftest.py                    # Pytest fixtures
│   ├── test_ingestion/
│   │   ├── test_pdf_parser.py
│   │   └── test_embedder.py
│   ├── test_tools/
│   │   ├── test_rag_query.py
│   │   └── test_gmp_scraper.py
│   ├── test_agents/
│   │   ├── test_data_extractor.py
│   │   └── test_quantitative_pulse.py
│   ├── test_orchestrator/
│   │   ├── test_state_management.py
│   │   └── test_synthesis.py
│   └── integration/
│       ├── test_cold_analysis.py
│       └── test_hot_loops.py
│
├── data/                              # Data storage (gitignored)
│   ├── input/                         # User-uploaded PDFs
│   ├── vector_store/                  # ChromaDB persistence
│   └── reports/                       # Generated reports
│
├── logs/                              # Application logs (gitignored)
│   └── ipo_analyst.log
│
└── scripts/                           # Utility scripts
    ├── init_database.py               # Initialize PostgreSQL schema
    ├── test_connections.py            # Verify all services
    └── seed_test_data.py              # Populate test IPO data
```

---

## 7. Development Roadmap

### Phase Overview

| Phase | Name | Duration Estimate | Key Deliverables |
|-------|------|-------------------|------------------|
| **Phase 0** | Foundation & Setup | 2-3 days | Project structure, Docker services, environment setup |
| **Phase 1** | Document Ingestion & RAG | 4-5 days | PDF parsing, ChromaDB integration, RAG query tool |
| **Phase 2** | Cold Analysis Agents | 5-6 days | Data Extractor, Validator, Fundamental Analyst, Risk Auditor |
| **Phase 3** | Hot Loop Agents | 6-7 days | Quantitative Pulse, Qualitative Research, web scraping |
| **Phase 4** | Orchestrator & State Management | 5-6 days | PostgreSQL persistence, Redis messaging, re-synthesis |
| **Phase 5** | Dual-Scheduler System | 3-4 days | APScheduler setup, market hours logic, event triggers |
| **Phase 6** | CLI & Reporting | 4-5 days | Typer commands, JSON/Markdown/HTML exporters |
| **Phase 7** | Testing & Integration | 5-6 days | Unit tests, integration tests, end-to-end validation |
| **Phase 8** | Polish & Documentation | 3-4 days | Code review, logging, error handling, README |

**Total Estimated Timeline:** 37-46 days (5-7 weeks at 8 hours/day)

**Document Version:** 2.0  
**Last Updated:** November 12, 2025  
**Maintained By:** IPO-Analyst-AGI Development Team
