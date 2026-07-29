# AI Voice Middleware Platform

An enterprise-grade, high-throughput, low-latency AI Voice Middleware Platform designed to sit between Speech-to-Text (STT) providers, Large Language Models (LLMs), Text-to-Speech (TTS) providers, and downstream Business APIs. 

Its primary objective is to make voice agents **Faster, Cheaper, More Reliable, Observable, and Adaptable**—initially focused on the Healthcare domain, with scalability built in for Banking, Government, and other enterprise verticals.

---

## 🌟 Product Philosophy: "LLM as a Last Resort"

We do not treat the LLM as the entry point or primary driver. Calling an LLM is slow, expensive, and unpredictable. The middleware employs a **deterministic-first** strategy, evaluating cheaper, faster, and more controlled options before routing to a generative model.

### Request Execution Flow:
1. **Detect Intent**: Classify input audio/text to determine the user's intent.
2. **Check Policy Engine**: Ensure the request complies with system policies (e.g., safety, system guardrails, or domain boundaries).
3. **Call Business API**: If the intent maps to a deterministic transaction (e.g., *lookup appointment*, *check patient file*), call the business API directly.
4. **Check Semantic Cache**: Scan for similar queries previously resolved and cached (e.g., FAQ retrieval) using vector embeddings.
5. **Build Optimized Context**: If an LLM call is required, build a trimmed, minimal prompt context using context compression (recent messages, short summary, specific memories).
6. **Select Appropriate LLM**: Route simple requests to cheaper, faster models (e.g., 8B/70B class) and complex queries to larger, reasoning models.
7. **Optimize Response**: Refine the output response, translate to target language if necessary, and clean output formatting for voice output.
8. **Convert to Speech**: Stream the output to the TTS provider for the user.

---

## 🏗️ Architecture: Clean Architecture (Ports & Adapters)

The codebase strictly adheres to **Hexagonal Architecture**. We separate pure business logic from outer technologies (like FastAPI, database dialects, or third-party API SDKs).

```
                      +-----------------------------+
                      |         Presentation        |
                      |  (FastAPI Routers, WS, HTTP)|
                      +--------------+--------------+
                                     |
                                     v
                      +--------------+--------------+
                      |         Application         |
                      |    (Use Cases, DTOs, Flows)  |
                      +--------------+--------------+
                                     |
                                     v
                      +--------------+--------------+
                      |            Domain           |
                      |  (Entities, Port Protocols) |
                      +--------------+--------------+
                                     ^
                                     |
                      +--------------+--------------+
                      |        Infrastructure       |
                      | (DB, Redis, LLM/TTS Clients)|
                      +-----------------------------+
```

### Decoupling Rules:
- **Domain Layer**: Contains no imports from external libraries or frameworks (no FastAPI, SQLAlchemy, or OpenAI SDK). Ports are defined as Python `Protocol` classes.
- **Application Layer**: Implements orchestrators and use-cases. Coordinates the flow of data using Domain Ports.
- **Infrastructure Layer**: Implements the concrete adapters (e.g., SQLAlchemy repositories, OpenAI client, Sarvam client).
- **Presentation Layer**: Handles incoming HTTP requests and WebSocket connections using FastAPI.

---

## 📁 Directory Structure

```text
app/
├── api/                  # Presentation: API endpoints and WebSocket handlers
│   └── v1/               # Versioned API routes (voice, chat, analytics, etc.)
├── application/          # Application: Orchestration and Use Cases
│   └── use_cases/        # Transactional workflows (e.g., process_voice_request)
├── core/                 # Core: Custom middleware reasoning and decision engines
│   ├── orchestrator.py   # Main pipeline orchestrator
│   ├── policy_engine.py  # Safety/rules engine
│   └── model_router.py   # Logic to route simple vs complex LLM calls
├── domain/               # Domain: Pure business rules, models, and port definitions
│   ├── models/           # Domain Entities (Patient, Session, AnalyticsMetric)
│   └── ports/            # Protocol definitions for all external interfaces (LLMPort, TTSPort, etc.)
├── infrastructure/       # Infrastructure: Concrete adapter implementations
│   ├── database/         # PostgreSQL, SQLAlchemy session management, Repositories
│   ├── cache/            # Redis semantic cache & context cache
│   └── providers/        # Third-party adapters (OpenAI, Sarvam, ElevenLabs, Whisper)
├── shared/               # Cross-cutting: Enums, Constants, Utilities
└── main.py               # Composition Root: Instantiates adapters and runs FastAPI
```

---

## 🛠️ Tech Stack & Setup

- **Language**: Python 3.13
- **Framework**: FastAPI (Async-first)
- **Database**: PostgreSQL (with `pgvector` for semantic search)
- **Cache**: Redis
- **ORM**: SQLAlchemy 2.0 (asyncio)

### Local Development Setup

1. **Virtual Environment**:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

2. **Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Environment Configuration**:
   Create a `.env` file in the root directory:
   ```env
   ENV=dev
   DEBUG=true
   DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/voice_middleware
   REDIS_URL=redis://localhost:6379/0
   
   # Provider Keys
   OPENAI_API_KEY=your-key-here
   SARVAM_API_KEY=your-key-here
   ELEVENLABS_API_KEY=your-key-here
   ```

4. **Run Application**:
   ```bash
   uvicorn app.main:app --reload
   ```
