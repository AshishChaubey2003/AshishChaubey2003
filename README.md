<h1 align="center">Hi, I'm Ashish Kumar Chaubey</h1>

<p align="center">
  <strong>Python Backend Developer &nbsp;|&nbsp; Django · FastAPI · PostgreSQL &nbsp;|&nbsp; GenAI Applications (RAG, Tool Calling, LangGraph Agents)</strong>
</p>

<p align="center">
  B.Tech Computer Science (2025) · Bengaluru, India · Open to Python Backend / Full-Stack / Junior AI Engineer roles · Immediate joiner
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/ashishchaubey2dec/"><img src="https://img.shields.io/badge/LinkedIn-ashishchaubey2dec-0A66C2?style=flat-square&logo=linkedin&logoColor=white" alt="LinkedIn"></a>
  <a href="mailto:ashish.chaubey.dev@gmail.com"><img src="https://img.shields.io/badge/Email-ashish.chaubey.dev%40gmail.com-EA4335?style=flat-square&logo=gmail&logoColor=white" alt="Email"></a>
  <a href="https://github.com/AshishChaubey2003?tab=repositories"><img src="https://img.shields.io/badge/GitHub-Repositories-181717?style=flat-square&logo=github&logoColor=white" alt="Repositories"></a>
</p>

<p align="center">
  <a href="#featured-projects">Projects</a> ·
  <a href="#engineering-highlights">Engineering Highlights</a> ·
  <a href="#technical-skills">Skills</a> ·
  <a href="#contact">Contact</a>
</p>

---

## About Me

I build backend systems in Python and connect them to real products: REST APIs with Django REST Framework and FastAPI, PostgreSQL data models that stay correct under concurrent writes, and LLM features that are grounded in the application's own data instead of free-form chat.

The problems I enjoy most sit at the boundary between a database and an API: making a booking flow safe against double-booking, verifying a payment webhook before trusting it, designing event pipelines that can be retried without side effects, and giving an LLM *tools* while keeping the backend, not the model, in charge of what actually gets written.

**Currently:** improving my deployed projects (TravelX, FreshPlate), extending a real-time voice sales agent (FastAPI + Pipecat + pgvector), and rebuilding TravelX on FastAPI + React/TypeScript.

---

## Featured Projects

Each project below links to source code. Live demos run on free tiers, so the first load can take 30–60 seconds while the service wakes up.

### 1. TravelX — travel booking backend with safe concurrent bookings and event-driven processing

Django REST Framework platform for booking hotels, adventures and vehicles across Indian cities, with Razorpay payments and a Kafka/Celery pipeline behind it.

**Stack:** Python · Django 5 · Django REST Framework · PostgreSQL · Kafka (confluent-kafka, transactional outbox) · Celery + Redis · Razorpay · JWT (SimpleJWT) · Docker Compose · Prometheus · Sentry · React + Leaflet frontend

**Key features**
- Email-OTP registration (OTPs stored as Argon2 hashes), JWT auth with refresh tokens, scoped rate limiting on login/OTP endpoints
- Hotel / adventure / vehicle catalog with filtering and pagination; booking create, list, detail and cancel
- Razorpay order creation, payment verification, webhooks and refunds
- Trip planner endpoint that builds a day-by-day plan from real listings for a chosen city
- Health endpoints (`/health/live`, `/health/ready`, `/health/dependencies`), JSON logs with request IDs, token-protected `/metrics`

**Engineering challenges solved**
- **No double-booking:** `select_for_update()` row locks inside `transaction.atomic()`, a date-overlap availability check, and `price_at_booking` snapshotting so later price edits never change what a customer was charged
- **Safe retries:** `Idempotency-Key` header on booking creation backed by a request fingerprint and a unique constraint
- **Trustworthy payments:** webhook signatures verified with HMAC-SHA256 and `hmac.compare_digest`
- **Reliable events:** transactional outbox → Kafka relay → idempotent consumer with a processed-event ledger, dead-letter table and a recovery command; a demo mode processes events inline when Kafka isn't available (used for the free-tier deployment)
- **Tests:** 500+ backend tests (pytest-django), including PostgreSQL-only concurrency tests

🔗 [Live demo](https://travelx-frontend-zdol.onrender.com) · [Source code](https://github.com/AshishChaubey2003/TravelX)

<details>
<summary>Screenshot</summary>
<br>
<img src="https://raw.githubusercontent.com/AshishChaubey2003/TravelX/main/screenshots/home.png" alt="TravelX home page" width="100%">
</details>

### 2. FreshPlate — cloud-kitchen ordering and food-rescue platform with a grounded AI assistant

Customers order freshly cooked meals and track delivery; donors list surplus food that an admin routes to rescue centres; a Gemini assistant answers questions using the live menu and the user's own orders.

**Stack:** Python · Django 6 · Django REST Framework · PostgreSQL · JWT (rotation + blacklisting) · Google Gemini (`google-genai`) · drf-spectacular (Swagger) · HTML/CSS/JavaScript frontend · Render + Vercel

**Key features**
- Three roles (customer, donor, admin) enforced with custom DRF permission classes
- Cart, order placement, order tracking and cancellation; admin order management
- Donation listings with a reviewed lifecycle (pending → approved → picked up → distributed)
- Chat assistant whose prompt is built from the real menu and the user's recent orders; conversation history and message length are capped
- Auto-generated OpenAPI docs at `/api/docs/`

**Engineering challenges solved**
- Order and donation status changes are governed by explicit state machines (`ALLOWED_TRANSITIONS`), so invalid transitions are rejected at the model layer
- Row locking (`select_for_update`) and `F()` expressions keep order totals and counters consistent under concurrent requests
- `select_related` / `Prefetch` / `annotate` plus database indexes to avoid N+1 queries on order and menu endpoints
- Scoped throttles for auth (10/min) and chatbot (20/min); production hardening with HSTS, SSL redirect and secure cookies
- Chatbot failures are wrapped in a dedicated `ChatbotUnavailable` error so the ordering flow never depends on the LLM

🔗 [Live demo](https://ai-fresshplate.vercel.app) · [Source code](https://github.com/AshishChaubey2003/Freshplate)

### 3. Voice Sales Agent — real-time voice assistant with hybrid RAG and consent-gated lead capture

A browser-based voice agent (WebRTC) that answers questions from a company's own documents and saves a visitor's contact details only after an explicit, verified "yes".

**Stack:** Python · FastAPI · Pipecat (WebRTC, Silero VAD) · Groq (LLM + Whisper STT) · Deepgram TTS · PostgreSQL 16 + pgvector · SQLAlchemy 2 (async) · Alembic · fastembed (`bge-small-en-v1.5`) · Docker

**Key features**
- Voice pipeline: transport → STT → per-turn knowledge injection → LLM → TTS, plus a text chat API with the same retrieval and tools
- Hybrid search: pgvector (HNSW) + PostgreSQL full-text (GIN) merged with Reciprocal Rank Fusion, with a distance gate that drops weak matches
- Multi-tenant data model: every table is scoped by `organization_id`
- Per-turn latency metrics (STT, LLM, TTS, voice-to-voice) stored per message, with a p50/p95 report script
- Concurrent-call cap and maximum call length; `/healthz` and `/readyz` endpoints

**Engineering challenges solved**
- **The LLM cannot save a lead on its own:** the `save_lead` tool is validated with Pydantic, requires that the assistant's previous message contained the email and that the visitor's last reply was an explicit confirmation, and the tenant/conversation IDs come from the server session, never from model output
- Settings use `SecretStr`, request schemas reject unknown fields, and retrieved documents are treated as data rather than instructions in the prompt
- 90+ pytest tests with fakes standing in for the LLM and search

🔗 [Source code](https://github.com/AshishChaubey2003/Voice-sales-agent) &nbsp;·&nbsp; *Runs locally (not deployed yet)*

### 4. Polyglot Code Review & Repair Agent — LLM review grounded in static analysis, with human approval

Reviews Python code for bugs, security and quality issues by combining deterministic analysis (AST, Ruff, Bandit) with parallel LLM review agents, and can propose a fix that is verified and only applied after a human approves it.

**Stack:** Python · LangGraph · LangChain · Groq · FAISS + BM25 (hybrid retrieval with RRF, reranking, optional HyDE) · sentence-transformers · Pydantic structured outputs · Streamlit UI · FastAPI API

**Key features**
- LangGraph pipeline: input guard → static analysis → retrieval → bug / security / quality agents in parallel → aggregate → optional fix → verification → report
- Review and repair available through both a Streamlit app and a FastAPI API (`/review`, `/repair`, `/verify`, `/approve`, `/apply`)
- Guardrails: size and encoding checks, prompt-injection phrase detection, secret scanning with masking, scope check that rejects fixes touching far more lines than the findings
- Small evaluation module computing precision / recall / F1 for the static analyzer and comparing vector-only vs hybrid vs hybrid+rerank retrieval

**Engineering challenges solved**
- Structured LLM output via Pydantic models with a JSON-mode fallback when tool-calling output fails validation
- Proposed fixes are re-run through syntax, lint and security checks before they can be approved; nothing is written to disk by the agent
- 130+ unit tests using fake embeddings and fake LLM responses

🔗 [Live demo (Streamlit)](https://polyglot-code-reviewer.streamlit.app/) · [Source code](https://github.com/AshishChaubey2003/polyglot-code-review-agent)

### 5. PolicyDesk AI — HR policy and leave assistant with backend-executed tool calls

One `/chat` endpoint behind JWT login: policy questions are answered by RAG over HR documents uploaded by an admin; leave balance and leave applications are handled by LLM tool calling where the backend runs the query.

**Stack:** Python · FastAPI · SQLAlchemy 2 · PostgreSQL · ChromaDB · Groq (Llama 3.3 70B) · PyJWT + bcrypt · pypdf · Docker Compose

**Key features**
- Register / login / `/me`, admin-only document upload (PDF or text) with chunking and re-ingestion
- Tools: `search_policy`, `get_leave_balance`, `apply_leave` (validates leave type, dates and remaining balance before writing)
- Tool loop capped at five rounds; system prompt restricts answers to retrieved policy text

**Engineering challenges solved**
- The user identity for every tool call comes from the JWT, so the model can never query or modify another employee's leave
- Deliberately framework-light (no LangChain/LangGraph) so every step of the RAG + tool loop is explicit and testable; 26 pytest tests run against SQLite without calling the LLM

🔗 [Source code](https://github.com/AshishChaubey2003/Policydesk-Ai)

<details>
<summary><strong>More projects</strong></summary>
<br>

- **[Gluzo AI](https://github.com/AshishChaubey2003/gluzo-ai)** — skincare recommendation chatbot: FastAPI + FAISS + sentence-transformers backend, React 18 + TypeScript + Tailwind frontend, routine builder and session memory
- **[PK Dosing Engine](https://github.com/AshishChaubey2003/Pk-dosing-engine)** — Terminal-Bench-style scientific task: two-compartment pharmacokinetic simulation (NumPy/SciPy) with parameter sensitivities, reference solution, weighted grader and Docker environment
- **[AI Code Reviewer](https://github.com/AshishChaubey2003/AI-Code-Reviewer)** — earlier Streamlit + LangGraph + FAISS reviewer, later rebuilt from scratch as the Polyglot agent above

</details>

---

## Engineering Highlights

What the repositories above demonstrate, with where to look:

| Area | What I've built |
|---|---|
| **REST API design** | Versioned DRF APIs with filtering, pagination and scoped throttling (TravelX, FreshPlate); FastAPI services with Pydantic v2 schemas that reject unknown fields (Voice Agent, PolicyDesk); auto-generated OpenAPI docs (FreshPlate) |
| **Authentication & authorization** | JWT with refresh-token rotation and blacklisting; role-based DRF permissions; email-OTP verification with hashed OTPs; bcrypt password hashing; identity taken from the token, never from request or model input |
| **Database modeling** | PostgreSQL schemas with explicit state machines, composite foreign keys for multi-tenancy, indexes and `select_related`/`Prefetch` query tuning; Alembic migrations in the FastAPI projects |
| **Concurrency & consistency** | `select_for_update()` inside atomic transactions, idempotency keys, price snapshots, `F()` counters, transactional outbox with idempotent consumers and dead-letter recovery |
| **Payments** | Razorpay order/verify/refund flow with HMAC-SHA256 webhook verification |
| **AI integration** | LLMs grounded in application data (menu + orders; policy documents; company knowledge base); tool calling where the backend validates and executes every action; Pydantic structured outputs with fallback parsing |
| **Retrieval** | pgvector + full-text hybrid search with RRF and a distance gate; FAISS + BM25 hybrid with reranking and HyDE; ChromaDB with chunk re-ingestion |
| **Agents & guardrails** | LangGraph graphs with parallel review nodes, human approval before any fix is applied, independent re-verification, prompt-injection and secret scanning |
| **Real-time voice** | Pipecat/WebRTC pipeline with VAD, STT, TTS and per-turn latency measurement (p50/p95 reporting) |
| **Error handling & observability** | Structured JSON logs with request IDs, Prometheus metrics, optional Sentry, health/readiness endpoints, graceful degradation when the LLM or a dependency is unavailable |
| **Automated testing** | pytest / pytest-django / pytest-asyncio suites: 500+ tests in TravelX, 130+ in the Polyglot agent, 90+ in the Voice Agent; fakes for LLMs and embeddings so tests run offline |
| **Frontend–backend integration** | React + Leaflet booking UI (TravelX), vanilla-JS fetch client with output escaping (FreshPlate), React + TypeScript + Vite + Tailwind with OpenAPI-generated types (TravelX v2, Gluzo) |
| **DevOps** | Docker and Docker Compose (Postgres, Redis, Kafka, workers), deployments on Render, Vercel and Streamlit Community Cloud with Neon PostgreSQL |

---

## Technical Skills

Technologies listed here are ones I have used in the projects above. Items marked *(learning)* are in progress.

**Programming Languages**

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat-square&logo=postgresql&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white)

**Backend Development**

![Django](https://img.shields.io/badge/Django-092E20?style=flat-square&logo=django&logoColor=white)
![Django REST Framework](https://img.shields.io/badge/Django_REST_Framework-A30000?style=flat-square&logo=django&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-D71F00?style=flat-square&logo=sqlalchemy&logoColor=white)
![Alembic](https://img.shields.io/badge/Alembic-6E6E6E?style=flat-square)
![Pydantic](https://img.shields.io/badge/Pydantic-E92063?style=flat-square&logo=pydantic&logoColor=white)
![Celery](https://img.shields.io/badge/Celery-37814A?style=flat-square&logo=celery&logoColor=white)
![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=flat-square&logo=apachekafka&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=flat-square&logo=jsonwebtokens&logoColor=white)
![REST APIs](https://img.shields.io/badge/REST_APIs-0052CC?style=flat-square)
![OpenAPI](https://img.shields.io/badge/OpenAPI_%2F_Swagger-85EA2D?style=flat-square&logo=swagger&logoColor=black)

**Frontend Development**

![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)
![React Router](https://img.shields.io/badge/React_Router-CA4245?style=flat-square&logo=reactrouter&logoColor=white)
![Leaflet](https://img.shields.io/badge/Leaflet-199900?style=flat-square&logo=leaflet&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white)

**Databases and Data Storage**

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![pgvector](https://img.shields.io/badge/pgvector-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat-square&logo=sqlite&logoColor=white)
![FAISS](https://img.shields.io/badge/FAISS-0467DF?style=flat-square&logo=meta&logoColor=white)
![ChromaDB](https://img.shields.io/badge/ChromaDB-FF6F00?style=flat-square)
![Neon](https://img.shields.io/badge/Neon_Postgres-00E599?style=flat-square&logo=neon&logoColor=black)

**AI and Generative AI**

![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=flat-square&logo=langgraph&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=flat-square&logo=langchain&logoColor=white)
![Groq](https://img.shields.io/badge/Groq_API-F55036?style=flat-square)
![Google Gemini](https://img.shields.io/badge/Google_Gemini-8E75B2?style=flat-square&logo=googlegemini&logoColor=white)
![RAG](https://img.shields.io/badge/RAG_%28hybrid_search,_RRF,_reranking%29-5C2D91?style=flat-square)
![LLM Tool Calling](https://img.shields.io/badge/LLM_Tool_Calling-5C2D91?style=flat-square)
![Pipecat](https://img.shields.io/badge/Pipecat_%28voice_%2F_WebRTC%29-2D2D2D?style=flat-square)
![Hugging Face](https://img.shields.io/badge/Sentence_Transformers_%2F_fastembed-FFD21E?style=flat-square&logo=huggingface&logoColor=black)
![Whisper](https://img.shields.io/badge/Whisper_STT-412991?style=flat-square&logo=openai&logoColor=white)
![Deepgram](https://img.shields.io/badge/Deepgram_TTS-13EF93?style=flat-square&logo=deepgram&logoColor=black)

**Developer Tools**

![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=flat-square&logo=docker&logoColor=white)
![Render](https://img.shields.io/badge/Render-000000?style=flat-square&logo=render&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)
![Ruff](https://img.shields.io/badge/Ruff-D7FF64?style=flat-square&logo=ruff&logoColor=black)
![Bandit](https://img.shields.io/badge/Bandit-FFC107?style=flat-square)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white)
![Sentry](https://img.shields.io/badge/Sentry-362D59?style=flat-square&logo=sentry&logoColor=white)
![VS Code](https://img.shields.io/badge/VS_Code-007ACC?style=flat-square&logo=visualstudiocode&logoColor=white)

**Testing and Evaluation**

![pytest](https://img.shields.io/badge/pytest-0A9EDC?style=flat-square&logo=pytest&logoColor=white)
![pytest-asyncio](https://img.shields.io/badge/pytest--asyncio-0A9EDC?style=flat-square&logo=pytest&logoColor=white)
![Django TestCase](https://img.shields.io/badge/Django_TestCase-092E20?style=flat-square&logo=django&logoColor=white)
![httpx](https://img.shields.io/badge/httpx_%2F_TestClient-2D2D2D?style=flat-square)
![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=flat-square&logo=vitest&logoColor=white)
![Testing Library](https://img.shields.io/badge/React_Testing_Library-E33332?style=flat-square&logo=testinglibrary&logoColor=white)
![MSW](https://img.shields.io/badge/MSW-FF6A33?style=flat-square)
![Evaluation](https://img.shields.io/badge/Retrieval_%26_analyzer_evaluation_%28P%2FR%2FF1%29-5C2D91?style=flat-square)

---

## GitHub Activity

<p align="center">
  <a href="https://github.com/AshishChaubey2003?tab=repositories">
    <img src="https://github-readme-stats.vercel.app/api?username=AshishChaubey2003&show_icons=true&hide_border=true&theme=default&include_all_commits=true&count_private=true" alt="GitHub stats" height="165">
  </a>
  <a href="https://github.com/AshishChaubey2003?tab=repositories">
    <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=AshishChaubey2003&layout=compact&hide_border=true&langs_count=8&hide=jupyter%20notebook" alt="Top languages" height="165">
  </a>
</p>

---

## Contact

| | |
|---|---|
| **LinkedIn** | [linkedin.com/in/ashishchaubey2dec](https://www.linkedin.com/in/ashishchaubey2dec/) |
| **Email** | [ashish.chaubey.dev@gmail.com](mailto:ashish.chaubey.dev@gmail.com) |
| **Live demos** | [TravelX](https://travelx-frontend-zdol.onrender.com) · [FreshPlate](https://ai-fresshplate.vercel.app) · [Polyglot Code Reviewer](https://polyglot-code-reviewer.streamlit.app/) |

<p align="center"><sub>Thanks for visiting. If you're hiring for a Python backend or GenAI role, the fastest way to see how I work is the TravelX and Voice Sales Agent repositories.</sub></p>
