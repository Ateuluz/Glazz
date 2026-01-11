**Project Roadmap & Development Pipeline**
=================================================

This roadmap provides a clear, practical pipeline for developing and completing a PDR-style RAG Chatbot project (backend + frontend) starting from a blank repository. It's written to be "idiot-safe": security and reproducibility are top priorities; a single task-runner (`just`) should encapsulate every multi-step developer action so contributors don't need to remember shell sequences.

Index
-----

1. Purpose & Principles
2. Prerequisites & Tooling
3. Initial repository setup
4. Development workflow (high-level)
5. Backend-focused pipeline
6. Frontend-focused pipeline
7. Testing strategy and checkpoints
8. CI / Release checklist
9. Security & secrets
10. Operational considerations (deploy, DB, vectors)
11. Checkpoints & milestones
12. Appendix: useful commands

1. Purpose & Principles
-----------------------

- Keep production safety, reproducibility, and incremental delivery central.
- Prefer small, testable increments; protect the data model and migrations.
- Automate repetitive tasks (dev env, linting, testing, migrations).
- Design APIs and user journeys before complex UI work to avoid rework.

Immediate first steps (do these before any code commit)
----------------------------------------------------

- Add a root `.gitignore` immediately and include entries that prevent secrets or local files from being committed (see Security section for example lines). Do this *before* creating any real `.env` or local files.
- Add a root `Justfile` with a minimal `setup` recipe and a `precommit` placeholder so contributors always run the same bootstrapping steps.
- Commit lockfiles where reproducibility is required (`uv.lock`, `package-lock.json`) unless you intentionally choose not to commit them — document this decision in the README.

Rationale: these three actions remove the biggest class of accidental leaks and platform differences early; make them the very first PR on a brand new repository.

2. Prerequisites & Tooling
--------------------------

Install and verify these tools locally (cross-platform focus):

- Git
- Docker & Docker Compose
- Node.js (v18+ / Node 20 recommended) — pin via `.nvmrc` or `engines`
- npm (bundled with Node)
- Python 3.13 (project targets 3.13)
- `uv` (environment/lock manager) — recommended for reproducible Python installs; repo will keep `uv.lock`
- `just` (task runner) — the single entrypoint for developer workflows (recommended)
- `psql` / `pg_isready` or run Postgres via Docker Compose (preferred for reproducible dev)
- Editor: VS Code with Python, ESLint/Prettier, and TypeScript support

Why each matters (short):
- `just` centralizes and documents all workflows so contributors don't type long command strings; use `just` as the primary interface (see sample recipes below).
- `uv` provides reproducible Python installs (via `uv.lock`) and maps to container workflow; it's optional but recommended for parity with builds.
- Docker ensures the local dev environment is identical to CI and production images (Postgres + `pgvector`).

3. Initial repository setup
---------------------------

1. If starting from an existing repo: inspect manifests: `pyproject.toml`, `uv.lock`, `package.json`, `docker-compose.yml` to understand declared dependencies.
 	If starting from scratch: request a project specification file that lists required language versions, desired package managers, and lockfile policy (this is the preferred input). If no specification file is provided, infer package manager/lockfile policy from context (presence of `pyproject.toml`, `package.json`, etc.) or fall back to conservative defaults — but do not unilaterally choose a package manager without confirmation. Treat dependency lists and package manager choices as a planning deliverable.
2. Start local DB via Docker Compose (recommended first run):

```bash
docker compose up -d db
```

3. Populate environment files (create or copy samples):

If templates exist, copy them; otherwise create minimal `.env` files with required keys. Example cross-platform guidance:

```bash
# If samples exist
test -f backend/.env.sample && cp backend/.env.sample backend/.env || echo "OPENAI_API_KEY=your-key-here" > backend/.env
test -f client/.env.sample && cp client/.env.sample client/.env || echo "VITE_API_BASE_URL=http://localhost:8000" > client/.env

# On Windows PowerShell (fallback create)
if (Test-Path backend\.env.sample) { Copy-Item backend\.env.sample backend\.env } else { Set-Content backend\.env 'OPENAI_API_KEY=your-key-here' }
if (Test-Path client\.env.sample) { Copy-Item client\.env.sample client\.env } else { Set-Content client\.env 'VITE_API_BASE_URL=http://localhost:8000' }
```

Edit the created files and *do not commit them*. Add a README snippet describing required env variables.

4. Install and sync Python dependencies using `uv` (required in this project's workflow):

```bash
pip install uv==0.9.22
uv sync --frozen
```

What `uv` guarantees and what it doesn't:
- `uv sync` installs packages from the lockfile into an isolated environment (not using system site-packages), producing deterministic installs from `uv.lock`.
- `uv` does not install or manage the system Python interpreter version. Ensure Python 3.13 is installed by the developer (use `pyenv`, Conda, or a system installer) and make that interpreter available on PATH before running `uv sync`.
- In short: `uv` ensures dependency reproducibility and isolation but not the system interpreter version; include `requires-python` in `pyproject.toml` and document the interpreter requirement in the roadmap.

5. Install frontend deps:

```bash
cd client
npm install
```

4. Development workflow (high-level)
-----------------------------------

- Start with the minimal valuable path (MVP) user flow: register → upload doc → ask chat question → receive cited response.
- Use API-first: design endpoints, data contracts (schemas), and example requests before UI.
- Build backend core behavior and tests first for the MVP path, then wire the frontend to the stable API.
- Iterate in small cycles: design → implement → tests → docs → demo.

Decision: Tests-first for core backend behavior
- For a project with DB migrations and RAG logic, prioritize tests-first for the backend core (auth, persistence, idempotency, document ingestion, vector storage, search). This reduces migration/data-loss risk and provides stable contracts for the frontend.

Justfile-first: recommended targets and cross-platform notes
---------------------------------------------------------

Make `just` the single developer entrypoint. Keep `Justfile` recipes short and prefer calling Python modules or Node scripts rather than embedding complex shell logic. This minimizes platform differences and keeps recipes readable.

Recommended `just` targets (implement at repo root):

- `setup` — installs Python tooling (`uv sync`), Node deps (`npm ci` or `npm install`), and sets up local DB containers.
- `db-up` / `db-down` — `docker compose up -d db` / `docker compose down`.
- `migrate:autogen` — `uv run python -m alembic revision --autogenerate -m "msg"`.
- `migrate` — `uv run python -m alembic upgrade head`.
- `backend:dev` — run backend dev server (`uv run uvicorn main:app --reload`) but call a Python helper so Windows can run it via `python -m` if needed.
- `frontend:dev` — `cd client && npm run dev`.
- `test` — `uv run python -m pytest` (unit tests) and `just test:integration` for integration tests.
 - `test` — runs the test sequence (unit then integration). Prefer explicit subtargets: `test:unit`, `test:integration`, `test:coverage`, `test:clean`.
- `lint` / `format` — `ruff` and `prettier` runs.
- `ci` — the same steps used in CI: install, lint, test, build.

Cross-platform guidance:

- `just` uses the user's shell; on Windows recommend using PowerShell or Git Bash. To avoid shell differences, keep heavy logic in Python scripts and invoke them from `just`.
- For anything that must run on both Windows and Unix, prefer `python -m package.module` or Node `node script.js` rather than complex shell one-liners.

Example: a `just setup` recipe should call `python -m scripts.bootstrap` which performs platform-specific steps internally in Python.

Can `just` reference other recipes or contents?
- Yes. `just` recipes can call other recipes by invoking `just <recipe>` (e.g., `ci` can call `setup` and `test`). Recipes can also call arbitrary shell commands or scripts. To avoid platform issues, prefer calling Python scripts (`python -m package.module`) from recipes instead of embedding shell-only logic.

5. Backend-focused pipeline
---------------------------

Start files and components
- `backend/app/model/` — define SQLAlchemy models first (users, documents, document_chunks, conversations, messages, idempotency)
- `backend/app/schemas/` — Pydantic request/response schemas to define API contracts
- `backend/app/api/` — routers for `auth`, `documents`, `conversations`, `chat`, `chat_stream`
- `backend/app/services/` — implement domain logic (document processing, embeddings, vector search, idempotency)
- `backend/alembic/versions/` — create safe migrations as models evolve

Initial backend steps
1. Write Pydantic schemas for the main API surface (auth, upload, chat request/response).
2. Implement SQLAlchemy models and run `alembic revision --autogenerate` to scaffold migrations.
3. Write unit tests for auth and CRUD using a temporary test DB (`pytest` + fixtures in `backend/tests/conftest.py`).
4. Implement document upload pipeline minimal: store file metadata, validate type/size, mark `pending` → `processing` → `ready`.
5. Add embeddings pipeline (mock embeddings in tests; integrate OpenAI or chosen model later).

Idempotency and consistency
- Implement `Idempotency-Key` header handling in upload endpoint before heavy processing.
- Store idempotency records and ensure retries map to same operation or return `409` when payload differs.

Streaming & performance
- Implement streaming chat as SSE or WebSocket early for API contract; allow non-streaming fallback.
- For vector search prefer `pgvector` + HNSW (as used in migrations) for reproducibility. Write an abstraction `app/services/vector_search.py` so alternative vector stores are pluggable.

6. Frontend-focused pipeline
---------------------------

Start files and components
- `client/src/pages/` — `Login.tsx`, `Chat.tsx`, `Landing.tsx`, `SignUp.tsx`
- `client/src/components/` — `MessageList`, `MessageInput`, `ConversationList`, `TopBar`, `DocumentIndicator`
- `client/src/services/` — `api.ts`, `auth.ts`, `chat.ts`, `document.ts` mapping to backend endpoints

Frontend steps
1. Implement auth flows and protected routing (`ProtectedRoute` already present). Test redirect behavior.
2. Implement minimal chat UI using mocked responses, then switch to real backend endpoints.
3. Implement document upload UI with idempotency-key generation (stable per file). Reuse behavior described in backend.
4. Add streaming UI that renders partial tokens/events as they arrive.

Styling & dev tools
- Tailwind + Prettier are already configured. Use Prettier as pre-commit hook and add `just format` or `npm run format` to tasks.

7. Testing strategy and checkpoints
---------------------------------

Testing pyramid & priorities
- Unit tests: services, helpers, data layer (fast). Start here.
- Integration tests: API endpoints with an ephemeral Postgres (Docker) and alembic migrations applied.
- End-to-end tests: simulate user flows (optional e2e tool or Playwright) for critical paths.

Suggested checkpoints and tests
- Checkpoint A: Auth + user lifecycle unit tests passing.
- Checkpoint B: Document upload pipeline with idempotency unit tests.
- Checkpoint C: Embedding storage and vector search (mocked) unit tests.
- Checkpoint D: API integration tests (pytest uses dockerized Postgres or testcontainers).

Testing commands

```bash
# Run backend unit tests
cd backend
python -m pytest tests/ -q

# Run frontend format check
cd client
npm run format -- --check
```

8. CI / Release checklist
-------------------------

CI pipeline steps (run on PRs):
- Lint + format checks (`ruff` for Python; Prettier for JS/TS)
- Install dependencies (use `uv sync` for Python step or plain pip install in CI if not using uv)
- Run unit tests
- Build frontend (`npm run build`) — optional on PRs, required on main
- Optional: run integration tests with a DB service

Release checklist (pre-deploy):
- Ensure migrations are finalized and included in repo
- Bump version and update `pyproject.toml`/`package.json` as needed
- Confirm environment variables and secrets storage for the environment
- Tag release and create changelog summary for the release

9. Security & secrets (top priority — do this first)
------------------------------------------------

Policy (must be enforced immediately on a new repo):

1) Do not commit secrets. Use local `.env` files for development only and never commit them. Use CI/production secret stores (GitHub/GitLab secrets, AWS Secrets Manager, etc.).

2) Add these `.gitignore` entries to the repository root immediately (commit this change before adding `.env` files):

```
# Environment / secrets
.env
.env.*.local
secrets.env
*.secret

# Python
.venv/
venv/
__pycache__/
.pytest_cache/

# Node
node_modules/
dist/
build/

# OS / editor
.DS_Store
.vscode/

# Lockfiles: commit uv.lock if you want reproducible installs. If so, DO NOT add uv.lock here.
# uv.lock
```

3) Commit lockfiles for reproducibility: prefer committing `uv.lock` and `package-lock.json` (or `pnpm-lock.yaml`) so environments are reproducible. Document your lockfile policy clearly in README.

4) Add pre-commit validation: use `pre-commit` hooks to run `ruff`, `prettier`, and a small check that prevents staging `.env` or other secret patterns.

5) Runtime safety: validate uploads (size/type), sanitize inputs, rate-limit expensive endpoints, and ensure strict authorization checks on all resource access.

6) Secrets in CI/Production: wire secrets only via the CI/CD secret facility or a secrets manager — never via files in the repo or plain environment dumps.

10. Operational considerations (deploy, DB, vectors)
-------------------------------------------------

- Postgres with `pgvector` is the default; ensure DB image matches production capabilities (extensions, indexes).
- Plan for reindexing/embedding updates: keep an indexed migration plan and scripts to backfill embeddings.
- Monitor latency of embedding calls and vector queries; add caching for repeated queries.

pgvector vs LangChain / LangGraph — compatibility and recommendation

- `pgvector` is a storage layer: a Postgres extension that stores vectors and provides ANN indexes (e.g., HNSW). It is not an orchestration or LLM helper library.
- `LangChain` and `LangGraph` are orchestration libraries that build chains, retrievers, prompt templates, and higher-level workflows. They expect a retriever/storage adapter beneath them.
- They are complementary: prefer storing vectors in `pgvector` for durability and ops simplicity, and use LangChain (or LangGraph) to orchestrate retrieval + LLM calls. Implement a small adapter (`app/services/vector_search.py`) that exposes the retrieval API LangChain expects (or use an existing pgvector connector) so the orchestration layer can plug in seamlessly.

Recommendation: use `pgvector` for storage + indexing, and LangChain for orchestration. Use LangGraph only if you need graph-style orchestration paradigms; otherwise LangChain + a pgvector adapter is simplest and most reproducible.

11. Checkpoints & milestones
---------------------------

- Milestone 1: Project skeleton + secure defaults (first commit)
	- Create root `.gitignore` that blocks `.env` and local secrets and add a `Justfile` with at least `setup` and `precommit` recipes. Commit lockfiles if you adopt them (`uv.lock`, `package-lock.json`).
- Milestone 2: Auth + basic CRUD endpoints implemented + unit tests passing.
- Milestone 3: Document ingestion + idempotency implemented + integration test.
- Milestone 4: Embeddings + vector search (pgvector) integrated + basic RAG responses.
- Milestone 5: Streaming endpoint + frontend integration + e2e smoke test.

12. Appendix: useful commands
----------------------------

Environment & init

```bash
# Start DB
docker compose up -d db

# Install python deps with uv (repo uses uv.lock)
pip install uv==0.9.22
uv sync --frozen

# Run backend locally (Justfile targets exist)
just backend

# Run frontend dev server
cd client
npm run dev
```

Database & migrations

```bash
# Create alembic revision (after model changes)
cd backend
uv run python -m alembic revision --autogenerate -m "describe change"

# Apply migrations
uv run python -m alembic upgrade head
```

Testing

```bash
# Run tests
cd backend
python -m pytest
```

Notes
-----

- Keep the API contracts (Pydantic schemas and frontend `types.d.ts`) in sync; changes to schemas should be small and accompanied by tests.
- Prefer small PRs that complete a single user story end-to-end (backend tests + frontend wiring + documentation).

---

This roadmap is intended as a living document. I can expand any section (example: a full CI YAML, a local Windows install guide, or a “how to run integration tests with Docker” section) on demand.

Suggested minimal `Justfile` (conceptual)
---------------------------------------

The following is an example structure to include in a `Justfile` at repo root. Prefer calling Python helpers from recipes to minimize shell differences.

```makefile
# Example Justfile (conceptual) — use Python helpers where possible to stay cross-platform

setup:
	@python -m scripts.bootstrap

db-up:
	docker compose up -d db

db-down:
	docker compose down

migrate:autogen:
	uv run python -m alembic revision --autogenerate -m "autogen"

migrate:
	uv run python -m alembic upgrade head

backend:dev:
	uv run uvicorn main:app --reload --host 0.0.0.0 --port 8000

frontend:dev:
	cd client && npm run dev

test:
	uv run python -m pytest -q

test:unit:
	uv run python -m pytest tests/unit -q

test:integration:
	# bring up a test DB, run migrations, run integration tests, then tear down
	docker compose up -d db
	uv run python -m alembic upgrade head
	uv run python -m pytest tests/integration -q
	docker compose down

test:clean:
	# cleanup test artifacts (databases, tmp files)
	python -m scripts.test_cleanup

ci:
	just setup
	just lint
	just test

```

Save `Justfile` at the repo root and keep recipes minimal — move platform-specific logic into `scripts/*.py`.
