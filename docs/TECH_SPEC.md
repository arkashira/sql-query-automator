# TECH_SPEC.md

## Project: sql-query-automator  
**Repository:** `arkashira/sql-query-automator`  
**Owner:** Axentx AI‑Workforce  
**Purpose:** Automate the creation, validation, and execution of ad‑hoc SQL queries using LLM inference, reducing manual effort for data engineers and analysts.

---

## 1. Architecture Overview

```
┌───────────────────────┐
│  Client (Web / CLI)   │
└─────────────┬─────────┘
              │
              ▼
┌───────────────────────┐
│  API Gateway (FastAPI)│
└───────┬───────────────┘
        │
        ▼
┌───────────────────────┐
│  Query Orchestrator    │
│  (Python, asyncio)     │
└───────┬───────────────┘
        │
        ├─► 1️⃣ Prompt Builder
        │     (Jinja2 templates + metadata)
        │
        ├─► 2️⃣ LLM Inference
        │     (vLLM + OpenAI GPT‑4o)
        │
        ├─► 3️⃣ Validation Engine
        │     (SQL parser + semantic checks)
        │
        └─► 4️⃣ Execution Engine
              (SQLAlchemy + asyncpg)
```

* **Front‑end**: Optional React SPA for interactive query building.  
* **Back‑end**: FastAPI microservice exposing REST endpoints.  
* **LLM**: vLLM inference engine running locally or via OpenAI API.  
* **Database**: PostgreSQL (or any SQL‑compatible DB) for query history, templates, and audit logs.  
* **Queue**: Redis for async task scheduling (Celery).  
* **Observability**: OpenTelemetry + Grafana Loki for tracing & logging.

---

## 2. Core Components

| Component | Responsibility | Key Libraries |
|-----------|----------------|---------------|
| **Prompt Builder** | Generates natural‑language prompts from user intent and metadata. | Jinja2, Pydantic |
| **LLM Inference** | Generates raw SQL from prompt. | vLLM, OpenAI SDK |
| **Validation Engine** | Parses SQL, checks for syntax, semantic correctness, and safety (e.g., no destructive statements). | sqlparse, sqlglot |
| **Execution Engine** | Executes safe queries, streams results, returns metadata. | SQLAlchemy, asyncpg |
| **Audit & History** | Stores query, intent, LLM output, validation status, and execution results. | SQLAlchemy ORM, Alembic |
| **Auth & Rate‑Limiting** | Secures endpoints and enforces per‑user limits. | FastAPI‑Users, Redis‑RateLimiter |
| **Metrics** | Exposes Prometheus metrics. | Prometheus‑Client |

---

## 3. Data Model

```sql
-- users
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    hashed_password TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- queries
CREATE TABLE queries (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    intent TEXT NOT NULL,
    prompt TEXT NOT NULL,
    raw_sql TEXT NOT NULL,
    validated BOOLEAN NOT NULL,
    validation_errors TEXT[],
    executed BOOLEAN,
    execution_error TEXT,
    result_metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- templates (optional)
CREATE TABLE templates (
    id UUID PRIMARY KEY,
    name TEXT UNIQUE NOT NULL,
    description TEXT,
    prompt_template TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);
```

*All timestamps are UTC.*

---

## 4. Key APIs / Interfaces

| Endpoint | Method | Description | Request Body | Response |
|----------|--------|-------------|--------------|----------|
| `/v1/queries/` | POST | Submit intent for a new query | `{"intent":"Show sales by region for Q1 2024"}` | `{"query_id":"uuid", "status":"queued"}` |
| `/v1/queries/{id}/status` | GET | Get status of a query | N/A | `{"status":"validated","executed":false}` |
| `/v1/queries/{id}/result` | GET | Retrieve executed results | N/A | `{"rows":[...], "columns":[...]}` |
| `/v1/templates/` | GET | List available templates | N/A | `[{id, name, description}]` |
| `/v1/templates/` | POST | Create a new template | `{"name":"SalesByRegion","prompt_template":"SELECT ... WHERE ..."} ` | `{"id":"uuid"}` |

**Authentication**: Bearer JWT via `Authorization: Bearer <token>`.

**Rate‑Limiting**: 60 queries/min per user.

---

## 5. Technology Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **API** | FastAPI | Modern async framework, automatic OpenAPI docs |
| **Inference** | vLLM (local) or OpenAI GPT‑4o | Fast, GPU‑accelerated, or cloud‑based LLM |
| **Database** | PostgreSQL | ACID, rich SQL support |
| **ORM** | SQLAlchemy + Alembic | Migrations, type safety |
| **Async Task Queue** | Celery + Redis | Decouple inference & execution |
| **Auth** | FastAPI‑Users + JWT | Standard, secure |
| **Observability** | OpenTelemetry, Prometheus, Grafana Loki | Metrics & logs |
| **Containerization** | Docker + Docker‑Compose | Reproducible dev & prod |
| **CI/CD** | GitHub Actions | Automated tests, linting, image build |
| **Testing** | pytest, httpx | Unit & integration tests |

---

## 6. Dependencies

| Category | Package | Version |
|----------|---------|---------|
| **Core** | fastapi | ^0.110.0 |
| | uvicorn | ^0.29.0 |
| | pydantic | ^2.6.0 |
| | sqlalchemy | ^2.0.30 |
| | asyncpg | ^0.29.0 |
| | alembic | ^1.13.0 |
| | redis | ^5.0.0 |
| | celery | ^5.4.0 |
| | jinja2 | ^3.1.4 |
| | sqlparse | ^0.4.4 |
| | sqlglot | ^20.0.0 |
| | vllm | ^0.5.0 |
| | openai | ^1.12.0 |
| **Auth** | fastapi-users | ^12.0.0 |
| **Observability** | opentelemetry-api | ^1.27.0 |
| | prometheus-client | ^0.20.0 |
| | loguru | ^0.7.2 |
| **Testing** | pytest | ^8.2.0 |
| | httpx | ^0.27.0 |
| | pytest-asyncio | ^0.23.0 |

*All dependencies are pinned to the latest stable releases at time of build.*

---

## 7. Deployment

### 7.1. Docker Compose (dev / prod)

```yaml
version: "3.9"
services:
  api:
    build: .
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/sql_automator
      - REDIS_URL=redis://redis:6379/0
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - db
      - redis
      - worker
  worker:
    build: .
    command: celery -A app.tasks worker --loglevel=info
    environment:
      - DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/sql_automator
      - REDIS_URL=redis://redis:6379/0
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    depends_on:
      - db
      - redis
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: sql_automator
    volumes:
      - db_data:/var/lib/postgresql/data
  redis:
    image: redis:7
    volumes:
      - redis_data:/data
volumes:
  db_data:
  redis_data:
```

### 7.2. Kubernetes (optional)

*Deploy the API, worker, DB, and Redis as separate Deployments/StatefulSets.*  
*Use Helm chart for easy rollout.*  
*Expose API via Ingress with TLS.*

### 7.3. CI/CD Pipeline

1. **Lint**: `ruff check .`  
2. **Test**: `pytest -n auto`  
3. **Build**: Docker image `ghcr.io/arkashira/sql-query-automator:latest`  
4. **Push**: to GitHub Container Registry  
5. **Deploy**: via GitHub Actions to staging cluster, then promote to prod after manual approval.

---

## 8. Security & Compliance

| Area | Mitigation |
|------|------------|
| **Authentication** | JWT with short expiry (15 min) + refresh token |
| **Input Validation** | Strict schema validation via Pydantic |
| **SQL Safety** | Reject destructive statements (`DROP`, `DELETE` without `WHERE`) |
| **Rate Limiting** | Redis‑based token bucket |
| **Secrets** | Stored in GitHub Secrets / Vault; never committed |
| **Audit** | All queries logged with user ID, timestamps, and LLM provenance |

---

## 9. Performance Targets

| Metric | Target |
|--------|--------|
| Prompt → LLM response | < 500 ms (local vLLM) |
| Validation | < 50 ms |
| Execution (small query) | < 1 s |
| Throughput | 30 queries/min per worker |
| Error Rate | < 0.5% |

---

## 10. Future Enhancements

1. **Multi‑tenant support** – separate schemas per customer.  
2. **Query caching** – Redis cache for identical queries.  
3. **Explain plan visualization** – integrate with pgAdmin.  
4. **Fine‑tuning** – custom LLM fine‑tuned on company data.  
5. **Auto‑suggestion UI** – incremental query building.

---

### Appendix

* **Repository Structure**  
  ```
  sql-query-automator/
  ├─ app/
  │  ├─ main.py
  │  ├─ api/
  │  ├─ services/
  │  ├─ models/
  │  ├─ tasks.py
  │  └─ ...
  ├─ tests/
  ├─ Dockerfile
  ├─ docker-compose.yml
  ├─ alembic/
  ├─ requirements.txt
  └─ README.md
  ```

* **License**: MIT (see LICENSE file).

---
