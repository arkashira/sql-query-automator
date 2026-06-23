# STORIES.md

## Product: sql-query-automator  
**Goal** – Automate ad‑hoc SQL query creation and management so that data engineers can focus on higher‑value tasks.  
**MVP** – A web UI + API that accepts natural‑language prompts, generates SQL, validates against a target schema, and stores reusable query templates.

---

## Epics & Story Backlog

| Epic | Story | Acceptance Criteria |
|------|-------|---------------------|
| **1. Prompt Intake & Context** | **1.1** As a data engineer, I want to submit a natural‑language prompt, so that the system can understand my query intent. | • UI text box accepts free‑text prompt.<br>• Prompt is sent to backend via POST `/api/query`.<br>• Backend logs prompt with timestamp and user ID. |
| | **1.2** As a data engineer, I want to specify the target database and schema, so that the AI can generate syntactically correct SQL. | • Prompt form includes dropdown for database.<br>• Selected schema is passed to the AI model.<br>• Backend validates schema exists in catalog. |
| | **1.3** As a data engineer, I want to attach optional constraints (e.g., date range, limit), so that the generated query meets my business rules. | • UI allows adding key‑value constraints.<br>• Constraints are merged into the prompt before AI call.<br>• Constraints are reflected in the final SQL. |
| **2. AI Generation & Validation** | **2.1** As a data engineer, I want the system to generate SQL using an LLM, so that I can quickly prototype queries. | • Backend calls `vLLM` endpoint with prompt + schema.<br>• Response contains `sql` string.<br>• SQL is returned to UI within 5 s. |
| | **2.2** As a data engineer, I want the generated SQL to be syntactically validated against the target schema, so that I avoid runtime errors. | • Backend runs `sqlparse` + `psycopg2` dry‑run.<br>• If syntax error, UI shows error message and raw SQL.<br>• If valid, UI shows success badge. |
| | **2.3** As a data engineer, I want the system to check for potential performance issues (e.g., missing indexes), so that my queries run efficiently. | • Backend runs simple static analysis (e.g., `EXPLAIN` with `ANALYZE=false`).<br>• If `SELECT *` or missing `WHERE`, UI shows warning. |
| **3. Query Management** | **3.1** As a data engineer, I want to save generated queries as templates, so that I can reuse them later. | • UI has “Save as Template” button.<br>• Template includes name, description, prompt, SQL, and schema.<br>• Template is stored in PostgreSQL table `query_templates`. |
| | **3.2** As a data engineer, I want to list, edit, and delete my templates, so that I can maintain a clean library. | • `/api/templates` GET returns paginated list.<br>• PUT `/api/templates/:id` updates name/description.<br>• DELETE `/api/templates/:id` removes template. |
| | **3.3** As a data engineer, I want to run a saved template with new parameters, so that I can quickly generate fresh results. | • UI allows selecting a template and overriding constraints.<br>• Backend re‑runs AI with updated prompt.<br>• Resulting SQL is displayed. |
| **4. Execution & Results** | **4.1** As a data engineer, I want to execute the generated SQL against the target database, so that I can see the results immediately. | • UI has “Run Query” button.<br>• Backend executes SQL via `psycopg2` and streams results.<br>• Results displayed in tabular format. |
| | **4.2** As a data engineer, I want to export query results to CSV/JSON, so that I can share them with stakeholders. | • UI offers “Export CSV” / “Export JSON” buttons.<br>• Export respects current pagination and filters. |
| **5. Security & Auditing** | **5.1** As a security officer, I want all executed queries to be logged, so that we can audit usage. | • Each execution logs user ID, timestamp, SQL, and target DB.<br>• Logs stored in `query_logs` table with immutable flag. |
| | **5.2** As a data engineer, I want role‑based access control, so that only authorized users can run queries. | • JWT auth with roles (`engineer`, `viewer`).<br>• API checks role before executing or saving queries. |
| **6. Documentation & Help** | **6.1** As a new user, I want inline help tips, so that I can learn how to use the tool. | • UI shows tooltip on prompt box explaining syntax.<br>• “Help” modal lists common prompts and examples. |
| | **6.2** As a data engineer, I want a FAQ page, so that I can troubleshoot common issues. | • `/docs/faq` page lists errors, solutions, and best practices. |
| **7. Monitoring & Metrics** | **7.1** As a product owner, I want to see usage metrics (queries per day, avg latency), so that I can assess adoption. | • `/api/metrics` returns JSON with counts and averages.<br>• Dashboard visualizes data in Grafana. |
| | **7.2** As a devops engineer, I want alerts on failed query executions, so that we can react quickly. | • Failed executions trigger Slack webhook with details. |

---

## MVP Release Order

1. **Prompt Intake & Context** (1.1‑1.3)  
2. **AI Generation & Validation** (2.1‑2.3)  
3. **Query Management** (3.1‑3.3)  
4. **Execution & Results** (4.1‑4.2)  
5. **Security & Auditing** (5.1‑5.2)  

All other epics (6‑7) are planned for Post‑MVP enhancements.

---

## Definition of Done (DoD)

- Unit tests ≥ 90 % coverage for each module.  
- End‑to‑end tests covering prompt → SQL → execution flow.  
- Code reviewed and merged via PR with CI passing.  
- Documentation updated (README, API docs, inline help).  
- Deployment to staging with smoke tests.  

---
