TOOLING INVENTORY
=================

Note: this file is produced by scanning repository files and manifests only.

1) All items (lexicographically sorted)

- Alembic
- App (backend/app package)
- Autoprefixer
- Axios
- Bcrypt
- Build system (Python: setuptools / wheel / pyproject.toml)
- Browser/HTML (index.html, debug_chat.html)
- Client (frontend folder)
- Conftest / pytest
- Curl
- Docker
- Docker Compose
- Dotenv (.env / python-dotenv)
- FAISS (mentioned in docs)
- FastAPI
- File formats present (.md, .py, .ts, .tsx, .js, .json, Dockerfile, .toml, .ini, .sh, .ps1)
- Frontend stack: React + Vite + TypeScript + Tailwind + PostCSS
- HTTPX
- Hosted images: node:20, python:3.13-slim, pgvector/pgvector:pg17
- Idempotency helper (backend service)
- ISort (import sorting referenced in ruff config)
- Just (Justfile / command runner)
- JS/TS tooling (vite, @vitejs/plugin-react, tsconfig)
- LangChain (langchain, langchain-openai, langchain-community)
- LangGraph
- Linux tooling in Dockerfile (wget used during image build)
- Local scripts (run_tests.sh, run_tests.ps1, quick_test.py, project_tests/*.ps1)
- Lockfiles: uv.lock (backend), package-lock.json (client)
- numpy
- Node (runtime for frontend; Dockerfile base)
- NPM (used in Dockerfile and scripts)
- OpenAI (openai python package + OPENAI_API_KEY env)
- Passlib
- pg_hba.conf / postgres data files (pgdata/)
- pgvector (Postgres extension and python package)
- pip (used in Dockerfile)
- pypdf
- pydantic / pydantic-settings
- python (language; pyproject target: 3.13)
- python-docx
- python-jose
- python-multipart
- Prettier
- PostCSS
- Project folders and notable structure (backend/, backend/app/, backend/alembic/, client/, client/src/, docs/, progress/, project_tests/, tests/, pgdata/)
- Puff: `uv` (environment/lock manager referenced via uv.lock and Dockerfile)
- Pwd: `uv.lock` (explicit lockfile present)
- Pydocstyle / ruff settings referenced
- React
- react-hot-toast
- Runtime: Uvicorn (uvicorn[standard])
- Runtime manager: uv (installed in backend Dockerfile; used via `uv run` in Justfile/Dockerfile)
- Routing / CORS (FastAPI CORSMiddleware)
- Ruff (configured in pyproject.toml)
- SQLAlchemy
- Scripts folder (backend/scripts)
- Server entry: `just backend-entrypoint` (Justfile entry used in Dockerfile)
- Shell / PowerShell helpers (.sh, .ps1 files)
- Tailwind CSS
- Testing: pytest and test files under backend/tests and project_tests
- Tools for embedding/vector search: pgvector, HNSW (referenced in migration/docs)
- TypeScript
- UUID (frontend dependency)
- Vite
- Vite plugin: @vitejs/plugin-react
- Windows scripts (.ps1)
- YAML / INI / TOML configs (docker-compose.yml, alembic.ini, pyproject.toml)
- Zip / packaging: setuptools, wheel


2) Categorized lists (each category sorted lexicographically)

Languages

- JavaScript
- Python (3.13)
- TypeScript

Frameworks / Application frameworks

- FastAPI
- React

Libraries (selected libs found in manifests and code)

- Alembic
- Axios
- LangChain
- LangGraph
- numpy
- openai
- pgvector (python package)
- pypdf
- python-docx
- python-jose
- python-multipart
- pydantic
- passlib
- sqlalchemy
- uvicorn

Dev / Build tools

- Autoprefixer
- Just
- NPM
- Prettier
- Ruff
- TypeScript compiler (tsc via devDependency)
- Vite
- @vitejs/plugin-react

CLI tools & external commands referenced in repo (alphabetical)

- curl
- docker
- docker-compose
- npm
- node
- pg_isready
- psql
- uv (environment/lock manager; `uv.lock` present and `uv` invoked in files)
- wget

Docker images referenced

- node:20
- pgvector/pgvector:pg17
- python:3.13-slim

Config & manifest files (found in repo)

- .env.sample (backend and client)
- .gitignore
- Dockerfile (backend)
- Dockerfile (client)
- docker-compose.yml
- package.json
- package-lock.json
- pyproject.toml
- tsconfig.json
- tsconfig.node.json
- vite.config.ts
- tailwind.config.js
- postcss.config.js
- alembic.ini
- uv.lock

Project structure (top-level folders)

- backend/
- client/
- docs/
- progress/
- project_tests/
- pgdata/
- tests/

Scripts and entrypoints

- Justfile (project task runner; used by Docker entrypoint)
- backend/run_tests.sh
- backend/run_tests.ps1
- backend/scripts/db_smoke.py
- various PowerShell test scripts under project_tests

Referenced but not declared in language manifests (items scanned in repo files but not listed as package deps in pyproject.toml or package.json)

- `uv` — referenced via `uv.lock`, `uv sync`, and `uv run` in Dockerfile/Justfile but `uv` is installed via `pip install` inside Dockerfile; `uv` is not listed as a top-level dependency in `pyproject.toml`.
- `just` — `Justfile` exists and the backend Dockerfile installs `just` binary; `just` is not a Python package and therefore not in `pyproject.toml`.
- `pg_isready`, `psql` — healthcheck and docs reference these Postgres client utilities (not Python packages).
- `npm` / `node` — frontend uses `package.json` and Dockerfile uses `node:20`; `node`/`npm` are external platform tools (not listed inside `package.json` as installable packages).
- `wget` — used in `backend/Dockerfile` to download `just` during image build (OS-level tool).
- `curl` — used extensively in docs for ad-hoc testing (not a repo dependency).

Notes and observations

- The repo includes `uv.lock` and Dockerfile instructions that install `uv` and then run `uv sync --frozen`; if you're running locally without `uv` installed, `uv` is the item you reported missing.
- `Justfile` is used as the project's command runner; local developers need `just` (OS-level tool) or can run the commands manually.
- Many commands in docs and `docker-compose.yml` (curl, pg_isready, psql) assume external CLI tools are available in the developer environment.