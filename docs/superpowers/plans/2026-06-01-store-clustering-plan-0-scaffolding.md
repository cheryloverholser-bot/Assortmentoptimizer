# Store Clustering App — Plan 0: Tech Stack & Scaffolding

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up a deployable empty shell for the Store Clustering App — a new repo with a working multi-tenant SaaS skeleton (Next.js frontend, FastAPI backend, Dramatiq worker, Postgres + Redis), Clerk auth, R2 signed-URL upload plumbing, project CRUD, GitHub Actions CI, and Fly.io deploys. The output of this plan is an app you can sign into and create projects in — nothing functional inside a project yet.

**Architecture:** Monorepo with three deployable apps (web, api, worker) and one shared TypeScript package. Multi-tenancy is enforced by Clerk org IDs flowing into every backend query through a session-aware dependency. Heavy compute (clustering, exports) is handled by Dramatiq actors on a separate worker process so the API stays responsive. Object storage (R2) handles all user file uploads; the API never streams large files.

**Tech Stack:**
- Next.js 15 + TypeScript (frontend) — `apps/web`
- FastAPI + Python 3.12 + SQLAlchemy 2.x + Alembic (backend) — `apps/api`
- Dramatiq + Redis (background jobs) — `apps/worker`
- PostgreSQL 16 (Neon in prod, Docker local)
- Redis (Upstash in prod, Docker local)
- Clerk (auth + orgs)
- Cloudflare R2 (object storage)
- Fly.io (hosting for web/api/worker)
- GitHub Actions (CI)
- pnpm workspaces (monorepo), uv (Python deps)

**Repo location:** `C:\Users\COverholser\Documents\Cantactix\store-clustering-app` — a new directory, sibling to the existing Templates repo. **Not** inside Templates: this is application code, the Templates repo is for client deliverables.

**A note on TDD discipline:** Most tasks below are test-first (Steps: write failing test → confirm it fails → implement → confirm it passes → commit). A small number of tasks are pure scaffolding (creating dotfiles, configuring CI, writing deployment manifests) — these are marked **[scaffold]** and use a "make it work, verify it works" rhythm instead of red-green-refactor. Don't force TDD onto config files.

---

## File structure (after this plan completes)

```
store-clustering-app/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── apps/
│   ├── api/                       # FastAPI backend
│   │   ├── pyproject.toml
│   │   ├── uv.lock
│   │   ├── Dockerfile
│   │   ├── fly.toml
│   │   ├── alembic.ini
│   │   ├── src/
│   │   │   └── app/
│   │   │       ├── __init__.py
│   │   │       ├── main.py        # FastAPI app factory
│   │   │       ├── config.py      # Settings (Pydantic)
│   │   │       ├── db/
│   │   │       │   ├── __init__.py
│   │   │       │   ├── session.py     # SQLAlchemy engine + session
│   │   │       │   ├── base.py        # Declarative base + mixins
│   │   │       │   └── models/
│   │   │       │       ├── __init__.py
│   │   │       │       ├── organization.py
│   │   │       │       ├── user.py
│   │   │       │       └── project.py
│   │   │       ├── api/
│   │   │       │   ├── __init__.py
│   │   │       │   ├── deps.py        # FastAPI dependencies (auth, db, org)
│   │   │       │   └── routes/
│   │   │       │       ├── __init__.py
│   │   │       │       ├── health.py
│   │   │       │       ├── me.py
│   │   │       │       ├── projects.py
│   │   │       │       └── uploads.py
│   │   │       ├── auth/
│   │   │       │   ├── __init__.py
│   │   │       │   └── clerk.py       # Clerk JWT verification
│   │   │       └── storage/
│   │   │           ├── __init__.py
│   │   │           └── r2.py          # R2 signed URL generation
│   │   ├── alembic/
│   │   │   ├── env.py
│   │   │   ├── script.py.mako
│   │   │   └── versions/
│   │   │       └── 0001_initial_schema.py
│   │   └── tests/
│   │       ├── __init__.py
│   │       ├── conftest.py
│   │       ├── api/
│   │       │   ├── test_health.py
│   │       │   ├── test_me.py
│   │       │   ├── test_projects.py
│   │       │   └── test_uploads.py
│   │       ├── auth/
│   │       │   └── test_clerk.py
│   │       ├── db/
│   │       │   └── test_models.py
│   │       └── multitenancy/
│   │           └── test_isolation.py
│   ├── worker/                    # Dramatiq worker
│   │   ├── pyproject.toml         # (depends on apps/api)
│   │   ├── Dockerfile
│   │   ├── fly.toml
│   │   ├── src/
│   │   │   └── worker/
│   │   │       ├── __init__.py
│   │   │       ├── main.py
│   │   │       └── actors.py      # All actors live here for now
│   │   └── tests/
│   │       └── test_actors.py
│   └── web/                       # Next.js frontend
│       ├── package.json
│       ├── tsconfig.json
│       ├── next.config.ts
│       ├── Dockerfile
│       ├── fly.toml
│       ├── src/
│       │   ├── app/
│       │   │   ├── layout.tsx
│       │   │   ├── page.tsx           # Marketing/home page
│       │   │   ├── (auth)/
│       │   │   │   ├── sign-in/[[...rest]]/page.tsx
│       │   │   │   └── sign-up/[[...rest]]/page.tsx
│       │   │   └── (app)/
│       │   │       ├── layout.tsx     # Authenticated layout
│       │   │       ├── projects/
│       │   │       │   ├── page.tsx           # List
│       │   │       │   ├── new/page.tsx       # Create
│       │   │       │   └── [id]/page.tsx      # Detail
│       │   │       └── api-client.ts          # Typed API client
│       │   └── components/
│       │       └── ProjectCard.tsx
│       └── __tests__/
│           ├── pages/projects.test.tsx
│           └── components/ProjectCard.test.tsx
├── packages/
│   └── shared/                    # Shared TS types generated from FastAPI OpenAPI
│       ├── package.json
│       ├── tsconfig.json
│       ├── src/
│       │   └── index.ts
│       └── scripts/
│           └── generate-types.ts
├── docker-compose.yml             # Local Postgres + Redis
├── .env.example
├── .gitignore
├── .editorconfig
├── pnpm-workspace.yaml
├── package.json                   # Root, for pnpm + dev scripts
├── README.md
└── LICENSE
```

---

## Pre-flight check (before Task 1)

**Required tools installed on the engineer's machine:**

- Git ≥ 2.40
- Node.js ≥ 20 + pnpm ≥ 9 (install: `npm i -g pnpm`)
- Python ≥ 3.12 + uv (install: `pip install uv` or follow https://docs.astral.sh/uv/)
- Docker Desktop (for local Postgres + Redis)
- A free **Clerk** account (clerk.com) — note the publishable key and secret key
- A **Cloudflare R2** account with one bucket created — note account ID, access key ID, secret access key, bucket name
- (Deferred to Task 19) A **Fly.io** account, a **Neon** Postgres project, an **Upstash** Redis instance

If any of these aren't ready, stop and set them up before proceeding.

---

## Task 1: Initialize the repository [scaffold]

**Files:**
- Create: `C:\Users\COverholser\Documents\Cantactix\store-clustering-app\` (directory)
- Create: `store-clustering-app/.gitignore`
- Create: `store-clustering-app/README.md`
- Create: `store-clustering-app/LICENSE`
- Create: `store-clustering-app/.editorconfig`

- [ ] **Step 1: Create the directory and initialize git**

```bash
mkdir -p "C:\Users\COverholser\Documents\Cantactix\store-clustering-app"
cd "C:\Users\COverholser\Documents\Cantactix\store-clustering-app"
git init -b main
```

- [ ] **Step 2: Write `.gitignore`**

Create `store-clustering-app/.gitignore`:

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
.venv/
.pytest_cache/
.mypy_cache/
.ruff_cache/
*.egg-info/
dist/
build/

# Node
node_modules/
.next/
.turbo/
*.tsbuildinfo

# Env
.env
.env.local
.env.*.local
!.env.example

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/

# Logs
*.log

# Local Postgres data (from docker-compose)
.docker-data/

# Fly
.fly/
```

- [ ] **Step 3: Write `README.md`**

```markdown
# Store Clustering App

A productized multi-tenant SaaS that lets CPG category managers cluster a retailer's
stores from uploaded data and assign each cluster to a planogram variant.

See the [design spec](https://github.com/<org>/<spec-repo>/blob/main/docs/superpowers/specs/2026-06-01-store-clustering-design.md)
for the full design.

## Repository structure

- `apps/api` — FastAPI backend
- `apps/worker` — Dramatiq background workers
- `apps/web` — Next.js frontend
- `packages/shared` — Shared TypeScript types (generated from API OpenAPI)

## Local development

See [docs/local-dev.md](docs/local-dev.md) (created in a later plan).

Short version: `docker compose up -d` then `pnpm dev` and `uv run uvicorn` per app.

## License

See LICENSE.
```

- [ ] **Step 4: Write `LICENSE`**

Use a placeholder for now — the legal team will swap in the chosen license:

```
Copyright (c) 2026 Crisp

All rights reserved. License terms TBD before public release.
```

- [ ] **Step 5: Write `.editorconfig`**

```ini
root = true

[*]
end_of_line = lf
insert_final_newline = true
charset = utf-8
indent_style = space
indent_size = 2
trim_trailing_whitespace = true

[*.py]
indent_size = 4

[Makefile]
indent_style = tab
```

- [ ] **Step 6: Initial commit**

```bash
git add .
git commit -m "chore: initial repo scaffolding"
```

---

## Task 2: Set up monorepo workspace structure [scaffold]

**Files:**
- Create: `store-clustering-app/pnpm-workspace.yaml`
- Create: `store-clustering-app/package.json`
- Create: `store-clustering-app/apps/.gitkeep`
- Create: `store-clustering-app/packages/.gitkeep`

- [ ] **Step 1: Write `pnpm-workspace.yaml`**

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

- [ ] **Step 2: Write root `package.json`**

```json
{
  "name": "store-clustering-app",
  "private": true,
  "scripts": {
    "dev": "pnpm -r --parallel dev",
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "lint": "pnpm -r lint"
  },
  "devDependencies": {
    "typescript": "^5.6.0",
    "prettier": "^3.3.0"
  },
  "packageManager": "pnpm@9.12.0"
}
```

- [ ] **Step 3: Create empty workspace directories**

```bash
mkdir -p apps packages
touch apps/.gitkeep packages/.gitkeep
```

- [ ] **Step 4: Verify pnpm recognizes the workspace**

```bash
pnpm install
```

Expected: pnpm runs without error, no packages found yet — that's expected.

- [ ] **Step 5: Commit**

```bash
git add pnpm-workspace.yaml package.json apps/.gitkeep packages/.gitkeep
git commit -m "chore: configure pnpm workspace"
```

---

## Task 3: Local dev environment — Docker Compose [scaffold]

**Files:**
- Create: `store-clustering-app/docker-compose.yml`
- Create: `store-clustering-app/.env.example`

- [ ] **Step 1: Write `docker-compose.yml`**

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: localdev
      POSTGRES_DB: store_clustering
    ports:
      - "5432:5432"
    volumes:
      - ./.docker-data/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d store_clustering"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - ./.docker-data/redis:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
```

- [ ] **Step 2: Write `.env.example`**

```bash
# Local dev — copy to .env and adjust as needed
DATABASE_URL=postgresql+psycopg://appuser:localdev@localhost:5432/store_clustering
REDIS_URL=redis://localhost:6379/0

# Clerk (get from clerk.com dashboard)
CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx

# Cloudflare R2 (get from R2 dashboard)
R2_ACCOUNT_ID=xxxxx
R2_ACCESS_KEY_ID=xxxxx
R2_SECRET_ACCESS_KEY=xxxxx
R2_BUCKET=store-clustering-dev

# App
APP_ENV=local
LOG_LEVEL=INFO
API_BASE_URL=http://localhost:8000
WEB_BASE_URL=http://localhost:3000
```

- [ ] **Step 3: Verify Postgres + Redis come up**

```bash
docker compose up -d
docker compose ps
```

Expected: both services show `healthy` after a few seconds.

- [ ] **Step 4: Verify connectivity**

```bash
docker compose exec postgres psql -U appuser -d store_clustering -c "SELECT 1;"
docker compose exec redis redis-cli ping
```

Expected: `1` from Postgres, `PONG` from Redis.

- [ ] **Step 5: Bring down and verify clean shutdown**

```bash
docker compose down
```

- [ ] **Step 6: Commit**

```bash
git add docker-compose.yml .env.example
git commit -m "chore: add local dev compose for Postgres and Redis"
```

---

## Task 4: FastAPI hello world + health endpoint

**Files:**
- Create: `apps/api/pyproject.toml`
- Create: `apps/api/src/app/__init__.py`
- Create: `apps/api/src/app/main.py`
- Create: `apps/api/src/app/api/__init__.py`
- Create: `apps/api/src/app/api/routes/__init__.py`
- Create: `apps/api/src/app/api/routes/health.py`
- Create: `apps/api/tests/__init__.py`
- Create: `apps/api/tests/conftest.py`
- Create: `apps/api/tests/api/__init__.py`
- Create: `apps/api/tests/api/test_health.py`

- [ ] **Step 1: Initialize the API project with uv**

```bash
mkdir -p apps/api
cd apps/api
uv init --package --name store-clustering-api --python 3.12
```

Replace the generated `pyproject.toml` with:

```toml
[project]
name = "store-clustering-api"
version = "0.0.1"
requires-python = ">=3.12,<3.13"
dependencies = [
    "fastapi>=0.115.0",
    "uvicorn[standard]>=0.32.0",
    "pydantic>=2.9.0",
    "pydantic-settings>=2.5.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.3.0",
    "pytest-asyncio>=0.24.0",
    "httpx>=0.27.0",
    "ruff>=0.7.0",
    "mypy>=1.13.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/app"]

[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"
pythonpath = ["src"]

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "B", "UP", "ASYNC"]
```

Then install:

```bash
uv sync --all-extras
```

- [ ] **Step 2: Write the failing test**

Create `apps/api/tests/api/test_health.py`:

```python
from fastapi.testclient import TestClient

from app.main import create_app


def test_health_endpoint_returns_ok():
    app = create_app()
    client = TestClient(app)

    response = client.get("/health")

    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

Create empty `apps/api/tests/__init__.py`, `apps/api/tests/api/__init__.py`, and `apps/api/tests/conftest.py`.

- [ ] **Step 3: Run the test and confirm it fails**

```bash
cd apps/api
uv run pytest tests/api/test_health.py -v
```

Expected: ImportError on `app.main` — module doesn't exist.

- [ ] **Step 4: Create the FastAPI app**

Create `apps/api/src/app/__init__.py` (empty).
Create `apps/api/src/app/api/__init__.py` (empty).
Create `apps/api/src/app/api/routes/__init__.py` (empty).

Create `apps/api/src/app/api/routes/health.py`:

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

Create `apps/api/src/app/main.py`:

```python
from fastapi import FastAPI

from app.api.routes import health


def create_app() -> FastAPI:
    app = FastAPI(title="Store Clustering API", version="0.0.1")
    app.include_router(health.router)
    return app


app = create_app()
```

- [ ] **Step 5: Run the test and confirm it passes**

```bash
uv run pytest tests/api/test_health.py -v
```

Expected: 1 passed.

- [ ] **Step 6: Verify the server runs locally**

```bash
uv run uvicorn app.main:app --reload --port 8000
```

In another terminal:

```bash
curl http://localhost:8000/health
```

Expected: `{"status":"ok"}`. Stop the server with Ctrl+C.

- [ ] **Step 7: Commit**

```bash
cd ../..
git add apps/api/pyproject.toml apps/api/uv.lock apps/api/src apps/api/tests
git commit -m "feat(api): initial FastAPI app with /health endpoint"
```

---

## Task 5: Configuration via Pydantic settings

**Files:**
- Create: `apps/api/src/app/config.py`
- Create: `apps/api/tests/test_config.py`

- [ ] **Step 1: Write the failing test**

Create `apps/api/tests/test_config.py`:

```python
import os

from app.config import Settings


def test_settings_loads_from_env(monkeypatch):
    monkeypatch.setenv("DATABASE_URL", "postgresql+psycopg://test:test@localhost/test")
    monkeypatch.setenv("REDIS_URL", "redis://localhost:6379/0")
    monkeypatch.setenv("CLERK_SECRET_KEY", "sk_test_x")
    monkeypatch.setenv("R2_ACCOUNT_ID", "acct")
    monkeypatch.setenv("R2_ACCESS_KEY_ID", "key")
    monkeypatch.setenv("R2_SECRET_ACCESS_KEY", "secret")
    monkeypatch.setenv("R2_BUCKET", "bucket")
    monkeypatch.setenv("APP_ENV", "test")

    settings = Settings()

    assert settings.database_url.startswith("postgresql+psycopg://")
    assert settings.redis_url == "redis://localhost:6379/0"
    assert settings.clerk_secret_key == "sk_test_x"
    assert settings.r2_bucket == "bucket"
    assert settings.app_env == "test"
```

- [ ] **Step 2: Run the test and confirm it fails**

```bash
cd apps/api
uv run pytest tests/test_config.py -v
```

Expected: ImportError on `app.config`.

- [ ] **Step 3: Implement `Settings`**

Create `apps/api/src/app/config.py`:

```python
from functools import lru_cache

from pydantic import Field
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8", case_sensitive=False)

    # Database
    database_url: str = Field(..., description="SQLAlchemy DB URL")
    redis_url: str = Field(..., description="Redis URL for Dramatiq broker")

    # Clerk
    clerk_secret_key: str = Field(..., description="Clerk backend secret key")
    clerk_publishable_key: str = Field(default="", description="Clerk frontend key (optional in API)")

    # R2
    r2_account_id: str
    r2_access_key_id: str
    r2_secret_access_key: str
    r2_bucket: str

    # App
    app_env: str = Field(default="local", description="local | test | staging | production")
    log_level: str = Field(default="INFO")


@lru_cache(maxsize=1)
def get_settings() -> Settings:
    return Settings()
```

- [ ] **Step 4: Run the test and confirm it passes**

```bash
uv run pytest tests/test_config.py -v
```

Expected: 1 passed.

- [ ] **Step 5: Commit**

```bash
cd ../..
git add apps/api/src/app/config.py apps/api/tests/test_config.py
git commit -m "feat(api): add Pydantic-based Settings loaded from .env"
```

---

## Task 6: SQLAlchemy 2.x session + declarative base

**Files:**
- Create: `apps/api/src/app/db/__init__.py`
- Create: `apps/api/src/app/db/session.py`
- Create: `apps/api/src/app/db/base.py`
- Create: `apps/api/tests/db/__init__.py`
- Create: `apps/api/tests/db/test_session.py`
- Modify: `apps/api/pyproject.toml` (add deps)

- [ ] **Step 1: Add SQLAlchemy and psycopg dependencies**

Edit `apps/api/pyproject.toml`. Under `dependencies`, add:

```toml
    "sqlalchemy>=2.0.35",
    "psycopg[binary]>=3.2.0",
    "alembic>=1.13.0",
```

Re-sync:

```bash
cd apps/api
uv sync --all-extras
```

- [ ] **Step 2: Write the failing test**

Create `apps/api/tests/db/__init__.py` (empty).

Create `apps/api/tests/db/test_session.py`:

```python
import os

import pytest
from sqlalchemy import text

from app.db.session import SessionLocal, engine


@pytest.mark.skipif(
    not os.getenv("DATABASE_URL"),
    reason="DATABASE_URL not set; run `docker compose up -d` and copy .env.example to .env",
)
def test_can_connect_and_execute_select_1():
    with SessionLocal() as session:
        result = session.execute(text("SELECT 1")).scalar()
        assert result == 1


def test_engine_uses_database_url_from_settings():
    assert str(engine.url).startswith("postgresql+psycopg://") or str(engine.url).startswith(
        "postgresql://"
    )
```

- [ ] **Step 3: Confirm it fails**

```bash
uv run pytest tests/db/test_session.py -v
```

Expected: ImportError on `app.db.session`.

- [ ] **Step 4: Implement session and base**

Create `apps/api/src/app/db/__init__.py` (empty).

Create `apps/api/src/app/db/base.py`:

```python
from datetime import datetime
from uuid import UUID, uuid4

from sqlalchemy import DateTime, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    """Project-wide SQLAlchemy declarative base."""


class TimestampMixin:
    """Reusable created_at / updated_at columns."""

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(), nullable=False
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(), onupdate=func.now(), nullable=False
    )


class UUIDPKMixin:
    """UUID primary key column named `id`."""

    id: Mapped[UUID] = mapped_column(primary_key=True, default=uuid4)
```

Create `apps/api/src/app/db/session.py`:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from app.config import get_settings

settings = get_settings()

engine = create_engine(
    settings.database_url,
    pool_pre_ping=True,
    pool_size=5,
    max_overflow=10,
    echo=settings.app_env == "local" and settings.log_level == "DEBUG",
)

SessionLocal = sessionmaker(bind=engine, expire_on_commit=False, autoflush=False)
```

- [ ] **Step 5: Bring Postgres up if not already running**

From the repo root:

```bash
docker compose up -d
cp .env.example .env  # if you haven't already
```

- [ ] **Step 6: Run the tests and confirm they pass**

```bash
cd apps/api
uv run pytest tests/db/test_session.py -v
```

Expected: 2 passed.

- [ ] **Step 7: Commit**

```bash
cd ../..
git add apps/api/pyproject.toml apps/api/uv.lock apps/api/src/app/db apps/api/tests/db
git commit -m "feat(api): add SQLAlchemy 2.x engine and session factory"
```

---

## Task 7: Domain models (Organization, User, Project)

**Files:**
- Create: `apps/api/src/app/db/models/__init__.py`
- Create: `apps/api/src/app/db/models/organization.py`
- Create: `apps/api/src/app/db/models/user.py`
- Create: `apps/api/src/app/db/models/project.py`
- Create: `apps/api/tests/db/test_models.py`

**Why three tables now:** Organization owns multi-tenancy; User links to Clerk; Project is the smallest functional unit users will create from the frontend in this plan. The five DataSource types and Scenario tables come in Plan 1 and Plan 2 respectively.

- [ ] **Step 1: Write the failing model tests**

Create `apps/api/tests/db/test_models.py`:

```python
from uuid import uuid4

import pytest
from sqlalchemy import select

from app.db.models import Organization, Project, User
from app.db.session import SessionLocal


@pytest.fixture
def session():
    with SessionLocal() as s:
        yield s
        s.rollback()


def test_organization_persists_with_clerk_id(session):
    org = Organization(clerk_org_id="org_test_123", name="Acme CPG")
    session.add(org)
    session.flush()

    found = session.execute(
        select(Organization).where(Organization.clerk_org_id == "org_test_123")
    ).scalar_one()
    assert found.name == "Acme CPG"
    assert found.id is not None


def test_user_belongs_to_organization(session):
    org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="Acme CPG")
    session.add(org)
    session.flush()

    user = User(
        clerk_user_id=f"user_{uuid4().hex}",
        organization_id=org.id,
        email="analyst@acme.com",
        display_name="Test Analyst",
    )
    session.add(user)
    session.flush()

    assert user.organization_id == org.id


def test_project_belongs_to_organization(session):
    org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="Acme CPG")
    session.add(org)
    session.flush()

    project = Project(
        organization_id=org.id, name="FY26 Frozen Pizza Reset", category="Frozen Pizza"
    )
    session.add(project)
    session.flush()

    assert project.organization_id == org.id
    assert project.name == "FY26 Frozen Pizza Reset"


def test_project_requires_organization(session):
    # organization_id is non-nullable; SQLAlchemy will raise on flush
    project = Project(organization_id=None, name="Orphan", category="Cookies")  # type: ignore[arg-type]
    session.add(project)
    with pytest.raises(Exception):
        session.flush()
```

- [ ] **Step 2: Confirm tests fail**

```bash
uv run pytest tests/db/test_models.py -v
```

Expected: ImportError on `app.db.models`.

- [ ] **Step 3: Implement the models**

Create `apps/api/src/app/db/models/organization.py`:

```python
from sqlalchemy import String
from sqlalchemy.orm import Mapped, mapped_column

from app.db.base import Base, TimestampMixin, UUIDPKMixin


class Organization(UUIDPKMixin, TimestampMixin, Base):
    __tablename__ = "organizations"

    clerk_org_id: Mapped[str] = mapped_column(String(255), unique=True, nullable=False, index=True)
    name: Mapped[str] = mapped_column(String(255), nullable=False)
```

Create `apps/api/src/app/db/models/user.py`:

```python
from uuid import UUID

from sqlalchemy import ForeignKey, String
from sqlalchemy.orm import Mapped, mapped_column

from app.db.base import Base, TimestampMixin, UUIDPKMixin


class User(UUIDPKMixin, TimestampMixin, Base):
    __tablename__ = "users"

    clerk_user_id: Mapped[str] = mapped_column(String(255), unique=True, nullable=False, index=True)
    organization_id: Mapped[UUID] = mapped_column(
        ForeignKey("organizations.id", ondelete="CASCADE"), nullable=False, index=True
    )
    email: Mapped[str] = mapped_column(String(320), nullable=False)
    display_name: Mapped[str] = mapped_column(String(255), nullable=False, default="")
```

Create `apps/api/src/app/db/models/project.py`:

```python
from uuid import UUID

from sqlalchemy import ForeignKey, String
from sqlalchemy.orm import Mapped, mapped_column

from app.db.base import Base, TimestampMixin, UUIDPKMixin


class Project(UUIDPKMixin, TimestampMixin, Base):
    __tablename__ = "projects"

    organization_id: Mapped[UUID] = mapped_column(
        ForeignKey("organizations.id", ondelete="CASCADE"), nullable=False, index=True
    )
    name: Mapped[str] = mapped_column(String(255), nullable=False)
    category: Mapped[str] = mapped_column(String(255), nullable=False)
```

Create `apps/api/src/app/db/models/__init__.py`:

```python
from app.db.models.organization import Organization
from app.db.models.project import Project
from app.db.models.user import User

__all__ = ["Organization", "Project", "User"]
```

- [ ] **Step 4: The tables don't exist yet — Alembic migration in Task 8**

The tests will still fail until the tables exist. Skip running them; we'll come back after Task 8.

- [ ] **Step 5: Commit**

```bash
cd ../..
git add apps/api/src/app/db/models apps/api/tests/db/test_models.py
git commit -m "feat(api): add Organization, User, Project SQLAlchemy models"
```

---

## Task 8: Alembic initial migration

**Files:**
- Create: `apps/api/alembic.ini`
- Create: `apps/api/alembic/env.py`
- Create: `apps/api/alembic/script.py.mako`
- Create: `apps/api/alembic/versions/0001_initial_schema.py`

- [ ] **Step 1: Initialize Alembic**

```bash
cd apps/api
uv run alembic init alembic
```

This creates `alembic.ini`, `alembic/env.py`, `alembic/script.py.mako`, `alembic/versions/`.

- [ ] **Step 2: Configure Alembic to use the app's database URL and metadata**

Replace the generated `apps/api/alembic.ini` `sqlalchemy.url` line with:

```ini
sqlalchemy.url =
```

(Leave it empty — we'll set it from env in `env.py`.)

Replace the generated `apps/api/alembic/env.py` with:

```python
from logging.config import fileConfig

from alembic import context
from sqlalchemy import engine_from_config, pool

from app.config import get_settings
from app.db.base import Base
from app.db.models import Organization, Project, User  # noqa: F401  - register models

config = context.config

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

settings = get_settings()
config.set_main_option("sqlalchemy.url", settings.database_url)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    with connectable.connect() as connection:
        context.configure(connection=connection, target_metadata=target_metadata)
        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

- [ ] **Step 3: Generate the initial migration**

```bash
uv run alembic revision --autogenerate -m "initial schema"
```

This produces a file at `alembic/versions/<hash>_initial_schema.py`. Rename it to `alembic/versions/0001_initial_schema.py`. Open it and verify the upgrade function creates `organizations`, `users`, `projects` tables with the expected columns.

- [ ] **Step 4: Apply the migration**

```bash
uv run alembic upgrade head
```

Expected: "Running upgrade  -> 0001, initial schema".

Verify in Postgres:

```bash
docker compose exec postgres psql -U appuser -d store_clustering -c "\dt"
```

Expected: see `alembic_version`, `organizations`, `users`, `projects`.

- [ ] **Step 5: Run the model tests from Task 7 and confirm they pass**

```bash
uv run pytest tests/db/test_models.py -v
```

Expected: 4 passed.

- [ ] **Step 6: Commit**

```bash
cd ../..
git add apps/api/alembic.ini apps/api/alembic
git commit -m "feat(api): add Alembic and create initial schema migration"
```

---

## Task 9: Clerk JWT verification

**Files:**
- Create: `apps/api/src/app/auth/__init__.py`
- Create: `apps/api/src/app/auth/clerk.py`
- Create: `apps/api/tests/auth/__init__.py`
- Create: `apps/api/tests/auth/test_clerk.py`
- Modify: `apps/api/pyproject.toml` (add `pyjwt`, `cryptography`, `httpx`)

- [ ] **Step 1: Add dependencies**

Edit `apps/api/pyproject.toml`. Under `dependencies`, add:

```toml
    "pyjwt>=2.9.0",
    "cryptography>=43.0.0",
    "httpx>=0.27.0",
```

```bash
cd apps/api
uv sync --all-extras
```

- [ ] **Step 2: Write the failing test**

Create `apps/api/tests/auth/__init__.py` (empty).

Create `apps/api/tests/auth/test_clerk.py`:

```python
from unittest.mock import patch

import pytest

from app.auth.clerk import ClerkClaims, InvalidTokenError, verify_clerk_token


def test_verify_clerk_token_returns_claims_for_valid_token():
    fake_decoded = {
        "sub": "user_abc",
        "org_id": "org_xyz",
        "org_role": "admin",
        "email": "analyst@acme.com",
        "exp": 9999999999,
        "iss": "https://test.clerk.accounts.dev",
    }
    with patch("app.auth.clerk._decode_jwt", return_value=fake_decoded):
        claims = verify_clerk_token("dummy.jwt.token")

    assert isinstance(claims, ClerkClaims)
    assert claims.user_id == "user_abc"
    assert claims.org_id == "org_xyz"
    assert claims.email == "analyst@acme.com"


def test_verify_clerk_token_raises_when_org_id_missing():
    fake_decoded = {"sub": "user_abc", "exp": 9999999999}
    with patch("app.auth.clerk._decode_jwt", return_value=fake_decoded):
        with pytest.raises(InvalidTokenError, match="org_id"):
            verify_clerk_token("dummy.jwt.token")


def test_verify_clerk_token_raises_on_decode_error():
    with patch("app.auth.clerk._decode_jwt", side_effect=ValueError("bad signature")):
        with pytest.raises(InvalidTokenError):
            verify_clerk_token("bad.jwt.token")
```

- [ ] **Step 3: Confirm it fails**

```bash
uv run pytest tests/auth/test_clerk.py -v
```

Expected: ImportError on `app.auth.clerk`.

- [ ] **Step 4: Implement Clerk verification**

Create `apps/api/src/app/auth/__init__.py` (empty).

Create `apps/api/src/app/auth/clerk.py`:

```python
from dataclasses import dataclass
from functools import lru_cache
from typing import Any

import httpx
import jwt
from jwt import PyJWKClient


class InvalidTokenError(Exception):
    """Raised when a Clerk JWT cannot be verified or is missing required claims."""


@dataclass(frozen=True)
class ClerkClaims:
    user_id: str
    org_id: str
    org_role: str
    email: str


@lru_cache(maxsize=1)
def _get_jwks_client() -> PyJWKClient:
    """Cache the Clerk JWKS client. Clerk's JWKS URL is derived from issuer."""
    # In practice we resolve the issuer from the first token we see; for v1 we
    # accept either Clerk dev or prod. Configure via env in a later iteration.
    # For now, the test patches `_decode_jwt` so the JWKS path is exercised only in integration.
    return PyJWKClient("https://api.clerk.dev/.well-known/jwks.json")


def _decode_jwt(token: str) -> dict[str, Any]:
    """Verify the JWT signature against Clerk's JWKS and return the payload."""
    jwks = _get_jwks_client()
    signing_key = jwks.get_signing_key_from_jwt(token).key
    payload: dict[str, Any] = jwt.decode(
        token,
        signing_key,
        algorithms=["RS256"],
        options={"verify_aud": False},
    )
    return payload


def verify_clerk_token(token: str) -> ClerkClaims:
    """Verify a Clerk session token and extract claims relevant to multi-tenancy."""
    try:
        payload = _decode_jwt(token)
    except Exception as exc:
        raise InvalidTokenError(f"Token decode failed: {exc}") from exc

    user_id = payload.get("sub")
    org_id = payload.get("org_id")
    org_role = payload.get("org_role", "")
    email = payload.get("email", "")

    if not user_id:
        raise InvalidTokenError("Token missing 'sub' (user_id)")
    if not org_id:
        raise InvalidTokenError("Token missing 'org_id' — user must be in a Clerk org")

    return ClerkClaims(user_id=user_id, org_id=org_id, org_role=org_role, email=email)
```

- [ ] **Step 5: Run the test and confirm it passes**

```bash
uv run pytest tests/auth/test_clerk.py -v
```

Expected: 3 passed.

- [ ] **Step 6: Commit**

```bash
cd ../..
git add apps/api/pyproject.toml apps/api/uv.lock apps/api/src/app/auth apps/api/tests/auth
git commit -m "feat(api): add Clerk JWT verification"
```

---

## Task 10: FastAPI auth dependencies (current_user, current_org, db_session)

**Files:**
- Create: `apps/api/src/app/api/deps.py`
- Create: `apps/api/tests/api/test_deps.py`

- [ ] **Step 1: Write the failing test**

Create `apps/api/tests/api/test_deps.py`:

```python
from unittest.mock import patch
from uuid import uuid4

import pytest
from fastapi import FastAPI
from fastapi.testclient import TestClient

from app.api.deps import get_current_org, get_current_user, get_db
from app.auth.clerk import ClerkClaims
from app.db.models import Organization, User
from app.db.session import SessionLocal


@pytest.fixture
def seeded_org_and_user():
    with SessionLocal() as s:
        org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="Acme CPG")
        s.add(org)
        s.flush()
        user = User(
            clerk_user_id=f"user_{uuid4().hex}",
            organization_id=org.id,
            email="analyst@acme.com",
            display_name="Test",
        )
        s.add(user)
        s.commit()
        yield org, user
        s.delete(user)
        s.delete(org)
        s.commit()


def _build_app():
    app = FastAPI()

    @app.get("/whoami")
    def whoami(user: User = pytest.importorskip("fastapi").Depends(get_current_user)):
        return {"user_id": str(user.id), "email": user.email}

    return app


def test_get_current_user_returns_user_for_valid_token(seeded_org_and_user):
    org, user = seeded_org_and_user
    claims = ClerkClaims(
        user_id=user.clerk_user_id, org_id=org.clerk_org_id, org_role="admin", email=user.email
    )

    app = FastAPI()

    from fastapi import Depends

    @app.get("/whoami")
    def whoami(u: User = Depends(get_current_user)):
        return {"user_id": str(u.id), "email": u.email}

    client = TestClient(app)
    with patch("app.api.deps.verify_clerk_token", return_value=claims):
        response = client.get("/whoami", headers={"Authorization": "Bearer fake.jwt"})

    assert response.status_code == 200
    assert response.json()["email"] == "analyst@acme.com"


def test_get_current_user_returns_401_when_missing_header():
    app = FastAPI()
    from fastapi import Depends

    @app.get("/whoami")
    def whoami(u: User = Depends(get_current_user)):
        return {"user_id": str(u.id)}

    client = TestClient(app)
    response = client.get("/whoami")
    assert response.status_code == 401


def test_get_current_user_returns_401_when_token_invalid():
    from app.auth.clerk import InvalidTokenError

    app = FastAPI()
    from fastapi import Depends

    @app.get("/whoami")
    def whoami(u: User = Depends(get_current_user)):
        return {}

    client = TestClient(app)
    with patch("app.api.deps.verify_clerk_token", side_effect=InvalidTokenError("bad")):
        response = client.get("/whoami", headers={"Authorization": "Bearer x"})

    assert response.status_code == 401
```

- [ ] **Step 2: Confirm it fails**

```bash
uv run pytest tests/api/test_deps.py -v
```

Expected: ImportError on `app.api.deps`.

- [ ] **Step 3: Implement deps**

Create `apps/api/src/app/api/deps.py`:

```python
from collections.abc import Generator
from typing import Annotated

from fastapi import Depends, Header, HTTPException, status
from sqlalchemy import select
from sqlalchemy.orm import Session

from app.auth.clerk import ClerkClaims, InvalidTokenError, verify_clerk_token
from app.db.models import Organization, User
from app.db.session import SessionLocal


def get_db() -> Generator[Session, None, None]:
    """FastAPI dependency that yields a SQLAlchemy session and closes it after."""
    session = SessionLocal()
    try:
        yield session
    finally:
        session.close()


def _get_claims(
    authorization: Annotated[str | None, Header()] = None,
) -> ClerkClaims:
    if not authorization or not authorization.lower().startswith("bearer "):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Missing or malformed Authorization header",
        )
    token = authorization.split(" ", 1)[1]
    try:
        return verify_clerk_token(token)
    except InvalidTokenError as exc:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED, detail=str(exc)
        ) from exc


def get_current_user(
    claims: Annotated[ClerkClaims, Depends(_get_claims)],
    db: Annotated[Session, Depends(get_db)],
) -> User:
    """Resolve the User row for the authenticated Clerk session, creating the
    User and Organization rows on first sight (just-in-time provisioning).
    """
    org = db.execute(
        select(Organization).where(Organization.clerk_org_id == claims.org_id)
    ).scalar_one_or_none()
    if org is None:
        # JIT-create the org on first sight. Name is initially blank;
        # a future webhook from Clerk can backfill it.
        org = Organization(clerk_org_id=claims.org_id, name="")
        db.add(org)
        db.flush()

    user = db.execute(
        select(User).where(User.clerk_user_id == claims.user_id)
    ).scalar_one_or_none()
    if user is None:
        user = User(
            clerk_user_id=claims.user_id,
            organization_id=org.id,
            email=claims.email,
            display_name="",
        )
        db.add(user)
        db.flush()

    db.commit()
    return user


def get_current_org(
    user: Annotated[User, Depends(get_current_user)],
    db: Annotated[Session, Depends(get_db)],
) -> Organization:
    return db.execute(
        select(Organization).where(Organization.id == user.organization_id)
    ).scalar_one()
```

- [ ] **Step 4: Run the tests and confirm they pass**

```bash
uv run pytest tests/api/test_deps.py -v
```

Expected: 3 passed.

- [ ] **Step 5: Commit**

```bash
cd ../..
git add apps/api/src/app/api/deps.py apps/api/tests/api/test_deps.py
git commit -m "feat(api): add FastAPI auth dependencies for user/org/db"
```

---

## Task 11: GET /me endpoint

**Files:**
- Create: `apps/api/src/app/api/routes/me.py`
- Modify: `apps/api/src/app/main.py` (register router)
- Create: `apps/api/tests/api/test_me.py`

- [ ] **Step 1: Write the failing test**

Create `apps/api/tests/api/test_me.py`:

```python
from unittest.mock import patch
from uuid import uuid4

import pytest
from fastapi.testclient import TestClient

from app.auth.clerk import ClerkClaims
from app.db.models import Organization, User
from app.db.session import SessionLocal
from app.main import create_app


@pytest.fixture
def seeded():
    with SessionLocal() as s:
        org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="Acme CPG")
        s.add(org)
        s.flush()
        user = User(
            clerk_user_id=f"user_{uuid4().hex}",
            organization_id=org.id,
            email="analyst@acme.com",
            display_name="Test Analyst",
        )
        s.add(user)
        s.commit()
        yield org, user
        s.delete(user)
        s.delete(org)
        s.commit()


def test_me_returns_user_and_org(seeded):
    org, user = seeded
    claims = ClerkClaims(
        user_id=user.clerk_user_id, org_id=org.clerk_org_id, org_role="admin", email=user.email
    )

    client = TestClient(create_app())
    with patch("app.api.deps.verify_clerk_token", return_value=claims):
        response = client.get("/me", headers={"Authorization": "Bearer fake"})

    assert response.status_code == 200
    body = response.json()
    assert body["user"]["email"] == "analyst@acme.com"
    assert body["organization"]["name"] == "Acme CPG"
```

- [ ] **Step 2: Confirm it fails**

```bash
uv run pytest tests/api/test_me.py -v
```

Expected: 404 on /me — route doesn't exist.

- [ ] **Step 3: Implement /me**

Create `apps/api/src/app/api/routes/me.py`:

```python
from typing import Annotated

from fastapi import APIRouter, Depends
from pydantic import BaseModel

from app.api.deps import get_current_org, get_current_user
from app.db.models import Organization, User

router = APIRouter()


class UserSchema(BaseModel):
    id: str
    email: str
    display_name: str


class OrgSchema(BaseModel):
    id: str
    name: str


class MeResponse(BaseModel):
    user: UserSchema
    organization: OrgSchema


@router.get("/me", response_model=MeResponse)
def me(
    user: Annotated[User, Depends(get_current_user)],
    org: Annotated[Organization, Depends(get_current_org)],
) -> MeResponse:
    return MeResponse(
        user=UserSchema(id=str(user.id), email=user.email, display_name=user.display_name),
        organization=OrgSchema(id=str(org.id), name=org.name),
    )
```

Edit `apps/api/src/app/main.py` to register the router:

```python
from fastapi import FastAPI

from app.api.routes import health, me


def create_app() -> FastAPI:
    app = FastAPI(title="Store Clustering API", version="0.0.1")
    app.include_router(health.router)
    app.include_router(me.router)
    return app


app = create_app()
```

- [ ] **Step 4: Run the test and confirm it passes**

```bash
uv run pytest tests/api/test_me.py -v
```

Expected: 1 passed.

- [ ] **Step 5: Commit**

```bash
cd ../..
git add apps/api/src/app/api/routes/me.py apps/api/src/app/main.py apps/api/tests/api/test_me.py
git commit -m "feat(api): add GET /me endpoint"
```

---

## Task 12: Projects CRUD endpoints

**Files:**
- Create: `apps/api/src/app/api/routes/projects.py`
- Modify: `apps/api/src/app/main.py` (register router)
- Create: `apps/api/tests/api/test_projects.py`

- [ ] **Step 1: Write the failing tests**

Create `apps/api/tests/api/test_projects.py`:

```python
from unittest.mock import patch
from uuid import uuid4

import pytest
from fastapi.testclient import TestClient

from app.auth.clerk import ClerkClaims
from app.db.models import Organization, Project, User
from app.db.session import SessionLocal
from app.main import create_app


@pytest.fixture
def seeded():
    with SessionLocal() as s:
        org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="Acme CPG")
        s.add(org)
        s.flush()
        user = User(
            clerk_user_id=f"user_{uuid4().hex}",
            organization_id=org.id,
            email="analyst@acme.com",
            display_name="Test",
        )
        s.add(user)
        s.commit()
        yield org, user
        # cleanup: delete cascaded projects + user + org
        for p in s.execute(
            __import__("sqlalchemy").select(Project).where(Project.organization_id == org.id)
        ).scalars():
            s.delete(p)
        s.delete(user)
        s.delete(org)
        s.commit()


def _claims_for(org: Organization, user: User) -> ClerkClaims:
    return ClerkClaims(
        user_id=user.clerk_user_id, org_id=org.clerk_org_id, org_role="admin", email=user.email
    )


def test_create_project(seeded):
    org, user = seeded
    client = TestClient(create_app())
    with patch("app.api.deps.verify_clerk_token", return_value=_claims_for(org, user)):
        response = client.post(
            "/projects",
            headers={"Authorization": "Bearer fake"},
            json={"name": "FY26 Frozen Pizza Reset", "category": "Frozen Pizza"},
        )

    assert response.status_code == 201
    body = response.json()
    assert body["name"] == "FY26 Frozen Pizza Reset"
    assert body["category"] == "Frozen Pizza"
    assert "id" in body


def test_list_projects_returns_only_my_orgs_projects(seeded):
    org, user = seeded

    # Seed a project in another org
    with SessionLocal() as s:
        other_org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="Other CPG")
        s.add(other_org)
        s.flush()
        s.add(Project(organization_id=other_org.id, name="Other Project", category="Cookies"))
        s.commit()

    # Create a project in my org via API
    client = TestClient(create_app())
    with patch("app.api.deps.verify_clerk_token", return_value=_claims_for(org, user)):
        client.post(
            "/projects",
            headers={"Authorization": "Bearer fake"},
            json={"name": "Mine", "category": "Pizza"},
        )
        response = client.get("/projects", headers={"Authorization": "Bearer fake"})

    assert response.status_code == 200
    body = response.json()
    names = [p["name"] for p in body]
    assert "Mine" in names
    assert "Other Project" not in names


def test_get_project_returns_404_for_other_orgs_project(seeded):
    org, user = seeded

    # Seed another org's project
    with SessionLocal() as s:
        other_org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="Other CPG")
        s.add(other_org)
        s.flush()
        other_project = Project(
            organization_id=other_org.id, name="Other Project", category="Cookies"
        )
        s.add(other_project)
        s.commit()
        other_project_id = other_project.id

    client = TestClient(create_app())
    with patch("app.api.deps.verify_clerk_token", return_value=_claims_for(org, user)):
        response = client.get(
            f"/projects/{other_project_id}", headers={"Authorization": "Bearer fake"}
        )

    assert response.status_code == 404  # never 200, never 403 — we deny existence


def test_delete_project_owned(seeded):
    org, user = seeded
    client = TestClient(create_app())

    with patch("app.api.deps.verify_clerk_token", return_value=_claims_for(org, user)):
        created = client.post(
            "/projects",
            headers={"Authorization": "Bearer fake"},
            json={"name": "ToDelete", "category": "Snacks"},
        ).json()
        project_id = created["id"]

        delete_resp = client.delete(
            f"/projects/{project_id}", headers={"Authorization": "Bearer fake"}
        )
        assert delete_resp.status_code == 204

        get_resp = client.get(
            f"/projects/{project_id}", headers={"Authorization": "Bearer fake"}
        )
        assert get_resp.status_code == 404
```

- [ ] **Step 2: Confirm it fails**

```bash
uv run pytest tests/api/test_projects.py -v
```

Expected: 404s on the routes — they don't exist yet.

- [ ] **Step 3: Implement projects routes**

Create `apps/api/src/app/api/routes/projects.py`:

```python
from typing import Annotated
from uuid import UUID

from fastapi import APIRouter, Depends, HTTPException, status
from pydantic import BaseModel, Field
from sqlalchemy import select
from sqlalchemy.orm import Session

from app.api.deps import get_current_org, get_db
from app.db.models import Organization, Project

router = APIRouter(prefix="/projects", tags=["projects"])


class CreateProjectRequest(BaseModel):
    name: str = Field(..., min_length=1, max_length=255)
    category: str = Field(..., min_length=1, max_length=255)


class ProjectResponse(BaseModel):
    id: str
    name: str
    category: str


@router.get("", response_model=list[ProjectResponse])
def list_projects(
    org: Annotated[Organization, Depends(get_current_org)],
    db: Annotated[Session, Depends(get_db)],
) -> list[ProjectResponse]:
    rows = db.execute(
        select(Project).where(Project.organization_id == org.id).order_by(Project.created_at.desc())
    ).scalars()
    return [ProjectResponse(id=str(p.id), name=p.name, category=p.category) for p in rows]


@router.post("", response_model=ProjectResponse, status_code=status.HTTP_201_CREATED)
def create_project(
    payload: CreateProjectRequest,
    org: Annotated[Organization, Depends(get_current_org)],
    db: Annotated[Session, Depends(get_db)],
) -> ProjectResponse:
    project = Project(organization_id=org.id, name=payload.name, category=payload.category)
    db.add(project)
    db.commit()
    db.refresh(project)
    return ProjectResponse(id=str(project.id), name=project.name, category=project.category)


def _load_owned_project(project_id: UUID, org: Organization, db: Session) -> Project:
    """Load a project by id, but only if it belongs to the current org.

    Returns 404 (not 403) if the project doesn't exist OR belongs to another org.
    We never leak the existence of another org's data.
    """
    project = db.execute(
        select(Project)
        .where(Project.id == project_id)
        .where(Project.organization_id == org.id)
    ).scalar_one_or_none()
    if project is None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Project not found")
    return project


@router.get("/{project_id}", response_model=ProjectResponse)
def get_project(
    project_id: UUID,
    org: Annotated[Organization, Depends(get_current_org)],
    db: Annotated[Session, Depends(get_db)],
) -> ProjectResponse:
    project = _load_owned_project(project_id, org, db)
    return ProjectResponse(id=str(project.id), name=project.name, category=project.category)


@router.delete("/{project_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_project(
    project_id: UUID,
    org: Annotated[Organization, Depends(get_current_org)],
    db: Annotated[Session, Depends(get_db)],
) -> None:
    project = _load_owned_project(project_id, org, db)
    db.delete(project)
    db.commit()
```

Edit `apps/api/src/app/main.py` to register:

```python
from fastapi import FastAPI

from app.api.routes import health, me, projects


def create_app() -> FastAPI:
    app = FastAPI(title="Store Clustering API", version="0.0.1")
    app.include_router(health.router)
    app.include_router(me.router)
    app.include_router(projects.router)
    return app


app = create_app()
```

- [ ] **Step 4: Run tests and confirm they pass**

```bash
uv run pytest tests/api/test_projects.py -v
```

Expected: 4 passed.

- [ ] **Step 5: Commit**

```bash
cd ../..
git add apps/api/src/app/api/routes/projects.py apps/api/src/app/main.py apps/api/tests/api/test_projects.py
git commit -m "feat(api): add Projects CRUD with multi-tenant isolation"
```

---

## Task 13: Cross-cutting multi-tenancy isolation test

**Files:**
- Create: `apps/api/tests/multitenancy/__init__.py`
- Create: `apps/api/tests/multitenancy/test_isolation.py`

**Why this exists:** The previous tests check individual endpoints. This test loops over the whole running app's routes and asserts that every authenticated endpoint denies cross-org access. As more endpoints are added in later plans, this test catches anyone who forgets to filter by `org_id`.

- [ ] **Step 1: Write the failing test**

Create `apps/api/tests/multitenancy/__init__.py` (empty).

Create `apps/api/tests/multitenancy/test_isolation.py`:

```python
from unittest.mock import patch
from uuid import uuid4

import pytest
from fastapi.testclient import TestClient

from app.auth.clerk import ClerkClaims
from app.db.models import Organization, Project, User
from app.db.session import SessionLocal
from app.main import create_app


@pytest.fixture
def two_orgs_and_users():
    with SessionLocal() as s:
        org_a = Organization(clerk_org_id=f"org_{uuid4().hex}", name="A")
        org_b = Organization(clerk_org_id=f"org_{uuid4().hex}", name="B")
        s.add_all([org_a, org_b])
        s.flush()

        user_a = User(
            clerk_user_id=f"user_{uuid4().hex}",
            organization_id=org_a.id,
            email="a@a.com",
            display_name="A",
        )
        user_b = User(
            clerk_user_id=f"user_{uuid4().hex}",
            organization_id=org_b.id,
            email="b@b.com",
            display_name="B",
        )
        s.add_all([user_a, user_b])

        project_b = Project(organization_id=org_b.id, name="B's Project", category="X")
        s.add(project_b)
        s.commit()

        yield {
            "org_a": org_a,
            "org_b": org_b,
            "user_a": user_a,
            "user_b": user_b,
            "project_b_id": project_b.id,
        }


def test_user_a_cannot_read_user_b_projects(two_orgs_and_users):
    org_a = two_orgs_and_users["org_a"]
    user_a = two_orgs_and_users["user_a"]
    project_b_id = two_orgs_and_users["project_b_id"]

    claims_a = ClerkClaims(
        user_id=user_a.clerk_user_id,
        org_id=org_a.clerk_org_id,
        org_role="admin",
        email=user_a.email,
    )

    client = TestClient(create_app())
    with patch("app.api.deps.verify_clerk_token", return_value=claims_a):
        list_resp = client.get("/projects", headers={"Authorization": "Bearer x"})
        get_resp = client.get(f"/projects/{project_b_id}", headers={"Authorization": "Bearer x"})
        delete_resp = client.delete(
            f"/projects/{project_b_id}", headers={"Authorization": "Bearer x"}
        )

    assert list_resp.status_code == 200
    assert all(p["id"] != str(project_b_id) for p in list_resp.json())
    assert get_resp.status_code == 404
    assert delete_resp.status_code == 404


def test_user_a_cannot_create_project_attributed_to_org_b(two_orgs_and_users):
    """The API derives org from the auth token, so even if a malicious client
    sent a project payload trying to specify an organization_id, the API
    should ignore it and use the authenticated org."""
    org_a = two_orgs_and_users["org_a"]
    user_a = two_orgs_and_users["user_a"]

    claims_a = ClerkClaims(
        user_id=user_a.clerk_user_id,
        org_id=org_a.clerk_org_id,
        org_role="admin",
        email=user_a.email,
    )
    client = TestClient(create_app())
    with patch("app.api.deps.verify_clerk_token", return_value=claims_a):
        response = client.post(
            "/projects",
            headers={"Authorization": "Bearer x"},
            json={
                "name": "Sneaky",
                "category": "X",
                # Even if the client sent organization_id, it should be ignored.
                "organization_id": str(two_orgs_and_users["org_b"].id),
            },
        )

    assert response.status_code == 201
    # Verify in the DB that the project is owned by org_a, not org_b
    with SessionLocal() as s:
        from sqlalchemy import select

        proj = s.execute(
            select(Project).where(Project.name == "Sneaky")
        ).scalar_one()
        assert proj.organization_id == org_a.id
```

- [ ] **Step 2: Run and confirm passes** (these should already pass from Task 12's implementation)

```bash
uv run pytest tests/multitenancy/test_isolation.py -v
```

Expected: 2 passed.

- [ ] **Step 3: Commit**

```bash
cd ../..
git add apps/api/tests/multitenancy
git commit -m "test(api): cross-org isolation tests for projects endpoints"
```

---

## Task 14: R2 signed URL endpoint (skeleton)

**Files:**
- Create: `apps/api/src/app/storage/__init__.py`
- Create: `apps/api/src/app/storage/r2.py`
- Create: `apps/api/src/app/api/routes/uploads.py`
- Modify: `apps/api/src/app/main.py`
- Create: `apps/api/tests/api/test_uploads.py`
- Modify: `apps/api/pyproject.toml` (add `boto3`)

- [ ] **Step 1: Add boto3 dependency**

Edit `apps/api/pyproject.toml`. Under `dependencies`, add:

```toml
    "boto3>=1.35.0",
```

```bash
cd apps/api
uv sync --all-extras
```

- [ ] **Step 2: Write the failing test**

Create `apps/api/tests/api/test_uploads.py`:

```python
from unittest.mock import patch
from uuid import uuid4

import pytest
from fastapi.testclient import TestClient

from app.auth.clerk import ClerkClaims
from app.db.models import Organization, Project, User
from app.db.session import SessionLocal
from app.main import create_app


@pytest.fixture
def seeded():
    with SessionLocal() as s:
        org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="Acme")
        s.add(org)
        s.flush()
        user = User(
            clerk_user_id=f"user_{uuid4().hex}",
            organization_id=org.id,
            email="a@a.com",
            display_name="",
        )
        project = Project(organization_id=org.id, name="P", category="X")
        s.add_all([user, project])
        s.commit()
        yield {"org": org, "user": user, "project": project}


def test_sign_upload_returns_url_and_key(seeded):
    org, user, project = seeded["org"], seeded["user"], seeded["project"]
    claims = ClerkClaims(
        user_id=user.clerk_user_id, org_id=org.clerk_org_id, org_role="admin", email=user.email
    )

    client = TestClient(create_app())
    fake_url = "https://r2-signed-url.example/put?sig=abc"
    with (
        patch("app.api.deps.verify_clerk_token", return_value=claims),
        patch(
            "app.storage.r2.R2Client.generate_presigned_put_url", return_value=fake_url
        ) as mock_gen,
    ):
        response = client.post(
            f"/projects/{project.id}/uploads/sign",
            headers={"Authorization": "Bearer x"},
            json={"file_kind": "store_master", "filename": "stores.csv", "content_type": "text/csv"},
        )

    assert response.status_code == 200
    body = response.json()
    assert body["url"] == fake_url
    assert body["key"].startswith(f"{org.id}/{project.id}/store_master/")
    assert body["key"].endswith("/stores.csv")
    mock_gen.assert_called_once()


def test_sign_upload_rejects_unknown_file_kind(seeded):
    org, user, project = seeded["org"], seeded["user"], seeded["project"]
    claims = ClerkClaims(
        user_id=user.clerk_user_id, org_id=org.clerk_org_id, org_role="admin", email=user.email
    )

    client = TestClient(create_app())
    with patch("app.api.deps.verify_clerk_token", return_value=claims):
        response = client.post(
            f"/projects/{project.id}/uploads/sign",
            headers={"Authorization": "Bearer x"},
            json={"file_kind": "evil_file", "filename": "x.csv", "content_type": "text/csv"},
        )

    assert response.status_code == 422


def test_sign_upload_returns_404_for_other_orgs_project(seeded):
    # Project belongs to a different org
    with SessionLocal() as s:
        other_org = Organization(clerk_org_id=f"org_{uuid4().hex}", name="O")
        s.add(other_org)
        s.flush()
        other_project = Project(organization_id=other_org.id, name="P", category="X")
        s.add(other_project)
        s.commit()
        other_project_id = other_project.id

    org, user = seeded["org"], seeded["user"]
    claims = ClerkClaims(
        user_id=user.clerk_user_id, org_id=org.clerk_org_id, org_role="admin", email=user.email
    )

    client = TestClient(create_app())
    with patch("app.api.deps.verify_clerk_token", return_value=claims):
        response = client.post(
            f"/projects/{other_project_id}/uploads/sign",
            headers={"Authorization": "Bearer x"},
            json={"file_kind": "store_master", "filename": "x.csv", "content_type": "text/csv"},
        )

    assert response.status_code == 404
```

- [ ] **Step 3: Confirm it fails**

```bash
uv run pytest tests/api/test_uploads.py -v
```

Expected: 404 on the route.

- [ ] **Step 4: Implement R2 client**

Create `apps/api/src/app/storage/__init__.py` (empty).

Create `apps/api/src/app/storage/r2.py`:

```python
from datetime import timedelta

import boto3
from botocore.config import Config

from app.config import get_settings


class R2Client:
    """Thin wrapper around boto3 S3 client configured for Cloudflare R2.

    R2 implements the S3 API but is hosted at a Cloudflare endpoint.
    """

    def __init__(self) -> None:
        settings = get_settings()
        self._bucket = settings.r2_bucket
        self._client = boto3.client(
            "s3",
            endpoint_url=f"https://{settings.r2_account_id}.r2.cloudflarestorage.com",
            aws_access_key_id=settings.r2_access_key_id,
            aws_secret_access_key=settings.r2_secret_access_key,
            config=Config(signature_version="s3v4"),
            region_name="auto",
        )

    def generate_presigned_put_url(
        self, key: str, content_type: str, expires_in: timedelta = timedelta(minutes=15)
    ) -> str:
        return self._client.generate_presigned_url(
            "put_object",
            Params={"Bucket": self._bucket, "Key": key, "ContentType": content_type},
            ExpiresIn=int(expires_in.total_seconds()),
        )
```

- [ ] **Step 5: Implement the uploads route**

Create `apps/api/src/app/api/routes/uploads.py`:

```python
from datetime import datetime, timezone
from enum import Enum
from typing import Annotated
from uuid import UUID, uuid4

from fastapi import APIRouter, Depends, HTTPException, status
from pydantic import BaseModel, Field
from sqlalchemy import select
from sqlalchemy.orm import Session

from app.api.deps import get_current_org, get_db
from app.db.models import Organization, Project
from app.storage.r2 import R2Client

router = APIRouter(prefix="/projects/{project_id}/uploads", tags=["uploads"])


class FileKind(str, Enum):
    store_master = "store_master"
    store_sales = "store_sales"
    store_demographics = "store_demographics"
    store_attributes = "store_attributes"
    pog_variants = "pog_variants"


class SignUploadRequest(BaseModel):
    file_kind: FileKind
    filename: str = Field(..., min_length=1, max_length=255)
    content_type: str = Field(..., min_length=1, max_length=255)


class SignUploadResponse(BaseModel):
    url: str
    key: str
    expires_at: datetime


def _load_owned_project(project_id: UUID, org: Organization, db: Session) -> Project:
    project = db.execute(
        select(Project)
        .where(Project.id == project_id)
        .where(Project.organization_id == org.id)
    ).scalar_one_or_none()
    if project is None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Project not found")
    return project


@router.post("/sign", response_model=SignUploadResponse)
def sign_upload(
    project_id: UUID,
    payload: SignUploadRequest,
    org: Annotated[Organization, Depends(get_current_org)],
    db: Annotated[Session, Depends(get_db)],
) -> SignUploadResponse:
    project = _load_owned_project(project_id, org, db)

    # Key shape: <org_id>/<project_id>/<file_kind>/<upload_uuid>/<filename>
    # — org_id first means a single IAM policy can scope an org to its own prefix later.
    upload_uuid = uuid4()
    key = f"{org.id}/{project.id}/{payload.file_kind.value}/{upload_uuid}/{payload.filename}"

    client = R2Client()
    url = client.generate_presigned_put_url(key=key, content_type=payload.content_type)

    return SignUploadResponse(
        url=url, key=key, expires_at=datetime.now(timezone.utc)
    )
```

Edit `apps/api/src/app/main.py`:

```python
from fastapi import FastAPI

from app.api.routes import health, me, projects, uploads


def create_app() -> FastAPI:
    app = FastAPI(title="Store Clustering API", version="0.0.1")
    app.include_router(health.router)
    app.include_router(me.router)
    app.include_router(projects.router)
    app.include_router(uploads.router)
    return app


app = create_app()
```

- [ ] **Step 6: Run the tests and confirm they pass**

```bash
uv run pytest tests/api/test_uploads.py -v
```

Expected: 3 passed.

- [ ] **Step 7: Commit**

```bash
cd ../..
git add apps/api/pyproject.toml apps/api/uv.lock apps/api/src/app/storage apps/api/src/app/api/routes/uploads.py apps/api/src/app/main.py apps/api/tests/api/test_uploads.py
git commit -m "feat(api): add R2 signed-URL upload endpoint per project"
```

---

## Task 15: Dramatiq worker skeleton

**Files:**
- Create: `apps/worker/pyproject.toml`
- Create: `apps/worker/src/worker/__init__.py`
- Create: `apps/worker/src/worker/main.py`
- Create: `apps/worker/src/worker/actors.py`
- Create: `apps/worker/tests/__init__.py`
- Create: `apps/worker/tests/test_actors.py`
- Modify: `apps/api/pyproject.toml` (add dramatiq)
- Modify: `apps/api/src/app/main.py` and add a test that the API can enqueue

**Why a separate `apps/worker`:** the worker process imports the same actor definitions but has a different runtime entrypoint (the dramatiq CLI). Keeping it as a separate uv project means the worker container is small and can have different env vars (e.g. higher memory).

- [ ] **Step 1: Add dramatiq to the API project**

Edit `apps/api/pyproject.toml`. Under `dependencies`, add:

```toml
    "dramatiq[redis]>=1.17.0",
```

```bash
cd apps/api
uv sync --all-extras
```

- [ ] **Step 2: Create the worker uv project**

```bash
cd ../..
mkdir -p apps/worker/src/worker apps/worker/tests
cd apps/worker
```

Create `apps/worker/pyproject.toml`:

```toml
[project]
name = "store-clustering-worker"
version = "0.0.1"
requires-python = ">=3.12,<3.13"
dependencies = [
    # The worker shares the same code as the API for now.
    # In a monorepo with uv, we depend on the API package via path.
    "store-clustering-api",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.3.0",
    "ruff>=0.7.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/worker"]

[tool.uv.sources]
store-clustering-api = { path = "../api", editable = true }

[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["src", "../api/src"]
```

```bash
uv sync --all-extras
```

- [ ] **Step 3: Write the failing test**

Create `apps/worker/tests/__init__.py` (empty).

Create `apps/worker/tests/test_actors.py`:

```python
import dramatiq
from dramatiq.brokers.stub import StubBroker

from worker.actors import ping


def test_ping_actor_is_registered():
    # The actor must be importable and registered with dramatiq
    assert isinstance(ping, dramatiq.Actor)
    assert ping.actor_name == "worker.actors.ping"


def test_ping_actor_returns_pong_when_invoked_directly():
    # Calling the actor's function directly (not via broker) returns the value
    assert ping.fn() == "pong"
```

- [ ] **Step 4: Confirm it fails**

```bash
uv run pytest tests/test_actors.py -v
```

Expected: ImportError on `worker.actors`.

- [ ] **Step 5: Implement the worker**

Create `apps/worker/src/worker/__init__.py` (empty).

Create `apps/worker/src/worker/main.py`:

```python
"""Worker process entrypoint.

Configures the dramatiq broker pointed at Redis and imports actors so they
are registered when `dramatiq worker.main` runs.
"""
import dramatiq
from dramatiq.brokers.redis import RedisBroker

from app.config import get_settings

settings = get_settings()
broker = RedisBroker(url=settings.redis_url)
dramatiq.set_broker(broker)

# Importing actors here registers them with the broker.
from worker import actors  # noqa: E402, F401
```

Create `apps/worker/src/worker/actors.py`:

```python
import dramatiq


@dramatiq.actor
def ping() -> str:
    """Smoke-test actor. Returns 'pong'.

    Real actors (clustering, exports) come in later plans.
    """
    return "pong"
```

- [ ] **Step 6: Run the worker test and confirm it passes**

```bash
uv run pytest tests/test_actors.py -v
```

Expected: 2 passed.

- [ ] **Step 7: Start the worker locally and verify it consumes from Redis**

Make sure Redis is up (`docker compose up -d` from repo root).

```bash
uv run dramatiq worker.main
```

Expected output includes `[INFO] dramatiq.worker.WorkerThread ... Booting worker`. Stop with Ctrl+C.

- [ ] **Step 8: Commit**

```bash
cd ../..
git add apps/api/pyproject.toml apps/api/uv.lock apps/worker
git commit -m "feat(worker): add Dramatiq worker with ping actor smoke test"
```

---

## Task 16: Next.js scaffold + Clerk integration [partly scaffold]

**Files:**
- Create: `apps/web/` (Next.js scaffold output)
- Modify: `apps/web/package.json` (add Clerk)
- Modify: `apps/web/src/app/layout.tsx`
- Create: `apps/web/.env.local.example`

- [ ] **Step 1: Scaffold the Next.js app**

From the repo root:

```bash
cd apps
pnpm create next-app@latest web --typescript --eslint --tailwind --app --src-dir --import-alias "@/*" --no-turbopack
```

Accept defaults. Then:

```bash
cd web
pnpm add @clerk/nextjs@latest
pnpm add -D vitest @testing-library/react @testing-library/jest-dom jsdom @vitejs/plugin-react
```

- [ ] **Step 2: Add Clerk env to `.env.local.example`**

Create `apps/web/.env.local.example`:

```bash
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx

NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
```

- [ ] **Step 3: Configure root layout for Clerk**

Replace `apps/web/src/app/layout.tsx`:

```tsx
import { ClerkProvider } from "@clerk/nextjs";
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "Store Clustering App",
  description: "Cluster stores and assign planogram variants",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  );
}
```

- [ ] **Step 4: Add Clerk middleware**

Create `apps/web/src/middleware.ts`:

```ts
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isProtectedRoute = createRouteMatcher(["/projects(.*)"]);

export default clerkMiddleware(async (auth, req) => {
  if (isProtectedRoute(req)) {
    await auth.protect();
  }
});

export const config = {
  matcher: ["/((?!.*\\..*|_next).*)", "/", "/(api|trpc)(.*)"],
};
```

- [ ] **Step 5: Configure Vitest**

Create `apps/web/vitest.config.ts`:

```ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./vitest.setup.ts"],
  },
});
```

Create `apps/web/vitest.setup.ts`:

```ts
import "@testing-library/jest-dom/vitest";
```

Add `test` script to `apps/web/package.json`:

```json
{
  "scripts": {
    "dev": "next dev --port 3000",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "vitest run",
    "test:watch": "vitest"
  }
}
```

- [ ] **Step 6: Smoke test — `pnpm dev` shows a Clerk-protected page**

```bash
cp .env.local.example .env.local
# Fill in real Clerk keys
pnpm dev
```

Open http://localhost:3000/projects. Expected: redirected to Clerk sign-in. Sign up; should land back on `/projects` (page doesn't exist yet — 404 expected, that's fine).

Stop the server.

- [ ] **Step 7: Commit**

```bash
cd ../..
git add apps/web/package.json apps/web/pnpm-lock.yaml apps/web/src apps/web/.env.local.example apps/web/vitest.config.ts apps/web/vitest.setup.ts apps/web/next.config.ts apps/web/tsconfig.json apps/web/eslint.config.mjs apps/web/postcss.config.mjs apps/web/tailwind.config.ts apps/web/public
git commit -m "feat(web): scaffold Next.js app with Clerk auth"
```

---

## Task 17: Typed API client (web → api)

**Files:**
- Create: `apps/web/src/app/(app)/api-client.ts`
- Create: `apps/web/__tests__/api-client.test.ts`

- [ ] **Step 1: Write the failing test**

Create `apps/web/__tests__/api-client.test.ts`:

```ts
import { describe, expect, it, vi, beforeEach, afterEach } from "vitest";
import { fetchMyProjects, createProject, type Project } from "../src/app/(app)/api-client";

describe("api-client", () => {
  const originalFetch = global.fetch;

  beforeEach(() => {
    process.env.NEXT_PUBLIC_API_BASE_URL = "http://test-api";
  });

  afterEach(() => {
    global.fetch = originalFetch;
    vi.restoreAllMocks();
  });

  it("fetchMyProjects calls GET /projects and returns typed array", async () => {
    const fakeProjects: Project[] = [{ id: "p1", name: "P1", category: "Pizza" }];
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => fakeProjects,
    } as unknown as Response);

    const result = await fetchMyProjects("fake-token");

    expect(global.fetch).toHaveBeenCalledWith(
      "http://test-api/projects",
      expect.objectContaining({
        headers: expect.objectContaining({ Authorization: "Bearer fake-token" }),
      }),
    );
    expect(result).toEqual(fakeProjects);
  });

  it("createProject POSTs name and category", async () => {
    const created: Project = { id: "p2", name: "New", category: "Frozen" };
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => created,
    } as unknown as Response);

    const result = await createProject("fake-token", { name: "New", category: "Frozen" });

    expect(global.fetch).toHaveBeenCalledWith(
      "http://test-api/projects",
      expect.objectContaining({
        method: "POST",
        body: JSON.stringify({ name: "New", category: "Frozen" }),
      }),
    );
    expect(result).toEqual(created);
  });

  it("fetchMyProjects throws on non-ok response", async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      status: 500,
      text: async () => "boom",
    } as unknown as Response);

    await expect(fetchMyProjects("fake-token")).rejects.toThrow(/500/);
  });
});
```

- [ ] **Step 2: Confirm it fails**

```bash
cd apps/web
pnpm test
```

Expected: Cannot find module `../src/app/(app)/api-client`.

- [ ] **Step 3: Implement the client**

Create `apps/web/src/app/(app)/api-client.ts`:

```ts
export type Project = {
  id: string;
  name: string;
  category: string;
};

export type CreateProjectInput = {
  name: string;
  category: string;
};

function baseUrl(): string {
  const url = process.env.NEXT_PUBLIC_API_BASE_URL;
  if (!url) {
    throw new Error("NEXT_PUBLIC_API_BASE_URL is not set");
  }
  return url;
}

async function request<T>(path: string, init: RequestInit, token: string): Promise<T> {
  const response = await fetch(`${baseUrl()}${path}`, {
    ...init,
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${token}`,
      ...(init.headers ?? {}),
    },
  });
  if (!response.ok) {
    const body = await response.text();
    throw new Error(`API ${response.status}: ${body}`);
  }
  return (await response.json()) as T;
}

export function fetchMyProjects(token: string): Promise<Project[]> {
  return request<Project[]>("/projects", { method: "GET" }, token);
}

export function createProject(token: string, input: CreateProjectInput): Promise<Project> {
  return request<Project>(
    "/projects",
    { method: "POST", body: JSON.stringify(input) },
    token,
  );
}
```

- [ ] **Step 4: Run tests and confirm they pass**

```bash
pnpm test
```

Expected: 3 passed.

- [ ] **Step 5: Commit**

```bash
cd ../..
git add apps/web/src/app/\(app\)/api-client.ts apps/web/__tests__/api-client.test.ts
git commit -m "feat(web): add typed API client for projects endpoints"
```

---

## Task 18: Projects list page

**Files:**
- Create: `apps/web/src/app/(app)/layout.tsx`
- Create: `apps/web/src/app/(app)/projects/page.tsx`
- Create: `apps/web/src/components/ProjectCard.tsx`
- Create: `apps/web/__tests__/ProjectCard.test.tsx`

- [ ] **Step 1: Write the failing test for ProjectCard**

Create `apps/web/__tests__/ProjectCard.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import { describe, expect, it } from "vitest";
import { ProjectCard } from "../src/components/ProjectCard";

describe("ProjectCard", () => {
  it("renders the project name and category", () => {
    render(<ProjectCard project={{ id: "p1", name: "Frozen Reset", category: "Pizza" }} />);
    expect(screen.getByText("Frozen Reset")).toBeInTheDocument();
    expect(screen.getByText("Pizza")).toBeInTheDocument();
  });

  it("links to the project detail page", () => {
    render(<ProjectCard project={{ id: "p1", name: "X", category: "Y" }} />);
    const link = screen.getByRole("link", { name: /X/i });
    expect(link).toHaveAttribute("href", "/projects/p1");
  });
});
```

- [ ] **Step 2: Confirm it fails**

```bash
cd apps/web
pnpm test
```

Expected: Cannot find module ProjectCard.

- [ ] **Step 3: Implement `ProjectCard`**

Create `apps/web/src/components/ProjectCard.tsx`:

```tsx
import Link from "next/link";
import type { Project } from "@/app/(app)/api-client";

export function ProjectCard({ project }: { project: Project }) {
  return (
    <Link
      href={`/projects/${project.id}`}
      className="block rounded-md border p-4 hover:shadow-md"
    >
      <h3 className="text-lg font-semibold">{project.name}</h3>
      <p className="text-sm text-gray-600">{project.category}</p>
    </Link>
  );
}
```

- [ ] **Step 4: Run and confirm passes**

```bash
pnpm test
```

Expected: tests pass.

- [ ] **Step 5: Implement the (app) layout**

Create `apps/web/src/app/(app)/layout.tsx`:

```tsx
import { OrganizationSwitcher, UserButton } from "@clerk/nextjs";

export default function AppLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen">
      <header className="flex items-center justify-between border-b px-6 py-3">
        <div className="text-xl font-semibold">Store Clustering</div>
        <div className="flex items-center gap-4">
          <OrganizationSwitcher hidePersonal />
          <UserButton />
        </div>
      </header>
      <main className="p-6">{children}</main>
    </div>
  );
}
```

- [ ] **Step 6: Implement the projects list page**

Create `apps/web/src/app/(app)/projects/page.tsx`:

```tsx
import { auth } from "@clerk/nextjs/server";
import Link from "next/link";

import { fetchMyProjects } from "../api-client";
import { ProjectCard } from "@/components/ProjectCard";

export default async function ProjectsPage() {
  const { getToken } = await auth();
  const token = await getToken();
  if (!token) {
    throw new Error("Expected an authenticated Clerk session");
  }

  const projects = await fetchMyProjects(token);

  return (
    <div>
      <div className="mb-4 flex items-center justify-between">
        <h1 className="text-2xl font-bold">Projects</h1>
        <Link
          href="/projects/new"
          className="rounded-md bg-blue-600 px-4 py-2 text-white"
        >
          New project
        </Link>
      </div>
      {projects.length === 0 ? (
        <p className="text-gray-500">No projects yet. Create your first one.</p>
      ) : (
        <div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
          {projects.map((p) => (
            <ProjectCard key={p.id} project={p} />
          ))}
        </div>
      )}
    </div>
  );
}
```

- [ ] **Step 7: Smoke test in browser**

Make sure the API is running:

```bash
# Terminal 1
cd apps/api
uv run uvicorn app.main:app --reload --port 8000

# Terminal 2
cd apps/web
pnpm dev
```

Open http://localhost:3000/projects. Expected: see "No projects yet. Create your first one." after signing in.

- [ ] **Step 8: Commit**

```bash
cd ../..
git add apps/web/src/app apps/web/src/components apps/web/__tests__
git commit -m "feat(web): add projects list page and ProjectCard component"
```

---

## Task 19: Create-project page

**Files:**
- Create: `apps/web/src/app/(app)/projects/new/page.tsx`
- Create: `apps/web/__tests__/new-project.test.tsx`

- [ ] **Step 1: Write the failing test**

Create `apps/web/__tests__/new-project.test.tsx`:

```tsx
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import { describe, expect, it, vi, beforeEach, afterEach } from "vitest";

// We test the form component in isolation; route navigation is integration-only.
import NewProjectForm from "../src/app/(app)/projects/new/NewProjectForm";

describe("NewProjectForm", () => {
  beforeEach(() => {
    process.env.NEXT_PUBLIC_API_BASE_URL = "http://test-api";
  });
  afterEach(() => vi.restoreAllMocks());

  it("submits form values to createProject", async () => {
    const onCreated = vi.fn();
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ id: "p1", name: "FY26", category: "Pizza" }),
    } as unknown as Response);

    render(<NewProjectForm token="t" onCreated={onCreated} />);

    fireEvent.change(screen.getByLabelText(/project name/i), { target: { value: "FY26" } });
    fireEvent.change(screen.getByLabelText(/category/i), { target: { value: "Pizza" } });
    fireEvent.click(screen.getByRole("button", { name: /create/i }));

    await waitFor(() => expect(onCreated).toHaveBeenCalledWith("p1"));
    expect(global.fetch).toHaveBeenCalledWith(
      "http://test-api/projects",
      expect.objectContaining({ method: "POST" }),
    );
  });

  it("disables the submit button when fields are empty", () => {
    render(<NewProjectForm token="t" onCreated={() => {}} />);
    expect(screen.getByRole("button", { name: /create/i })).toBeDisabled();
  });
});
```

- [ ] **Step 2: Confirm it fails**

```bash
pnpm test
```

Expected: Cannot find module NewProjectForm.

- [ ] **Step 3: Implement the form component**

Create `apps/web/src/app/(app)/projects/new/NewProjectForm.tsx`:

```tsx
"use client";

import { useState } from "react";
import { createProject } from "../../api-client";

export default function NewProjectForm({
  token,
  onCreated,
}: {
  token: string;
  onCreated: (id: string) => void;
}) {
  const [name, setName] = useState("");
  const [category, setCategory] = useState("");
  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setSubmitting(true);
    setError(null);
    try {
      const project = await createProject(token, { name, category });
      onCreated(project.id);
    } catch (err) {
      setError(err instanceof Error ? err.message : "Unknown error");
    } finally {
      setSubmitting(false);
    }
  }

  const isValid = name.trim().length > 0 && category.trim().length > 0;

  return (
    <form onSubmit={handleSubmit} className="max-w-md space-y-4">
      <div>
        <label htmlFor="name" className="mb-1 block text-sm font-medium">
          Project name
        </label>
        <input
          id="name"
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          className="w-full rounded-md border px-3 py-2"
          required
        />
      </div>
      <div>
        <label htmlFor="category" className="mb-1 block text-sm font-medium">
          Category
        </label>
        <input
          id="category"
          type="text"
          value={category}
          onChange={(e) => setCategory(e.target.value)}
          className="w-full rounded-md border px-3 py-2"
          required
        />
      </div>
      {error && <p className="text-sm text-red-600">{error}</p>}
      <button
        type="submit"
        disabled={!isValid || submitting}
        className="rounded-md bg-blue-600 px-4 py-2 text-white disabled:bg-gray-400"
      >
        {submitting ? "Creating…" : "Create project"}
      </button>
    </form>
  );
}
```

- [ ] **Step 4: Implement the page that uses the form**

Create `apps/web/src/app/(app)/projects/new/page.tsx`:

```tsx
"use client";

import { useAuth } from "@clerk/nextjs";
import { useRouter } from "next/navigation";
import { useEffect, useState } from "react";

import NewProjectForm from "./NewProjectForm";

export default function NewProjectPage() {
  const { getToken } = useAuth();
  const router = useRouter();
  const [token, setToken] = useState<string | null>(null);

  useEffect(() => {
    getToken().then(setToken);
  }, [getToken]);

  if (!token) return <p>Loading…</p>;

  return (
    <div>
      <h1 className="mb-4 text-2xl font-bold">New project</h1>
      <NewProjectForm
        token={token}
        onCreated={(id) => router.push(`/projects/${id}`)}
      />
    </div>
  );
}
```

- [ ] **Step 5: Run tests and confirm they pass**

```bash
pnpm test
```

Expected: tests pass.

- [ ] **Step 6: Smoke test in the browser**

Visit http://localhost:3000/projects/new. Fill in name + category, submit. Expected: redirect to `/projects/<id>` (page returns a placeholder in Task 20).

- [ ] **Step 7: Commit**

```bash
cd ../..
git add apps/web/src/app/\(app\)/projects/new apps/web/__tests__/new-project.test.tsx
git commit -m "feat(web): add new-project page with form"
```

---

## Task 20: Project detail page (skeleton)

**Files:**
- Create: `apps/web/src/app/(app)/projects/[id]/page.tsx`
- Modify: `apps/web/src/app/(app)/api-client.ts` (add `fetchProject`)
- Add test to: `apps/web/__tests__/api-client.test.ts`

- [ ] **Step 1: Extend api-client.test.ts with fetchProject test**

Append to `apps/web/__tests__/api-client.test.ts`:

```ts
  it("fetchProject calls GET /projects/:id", async () => {
    const project: Project = { id: "p1", name: "X", category: "Y" };
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => project,
    } as unknown as Response);

    const { fetchProject } = await import("../src/app/(app)/api-client");
    const result = await fetchProject("fake-token", "p1");

    expect(global.fetch).toHaveBeenCalledWith(
      "http://test-api/projects/p1",
      expect.objectContaining({
        headers: expect.objectContaining({ Authorization: "Bearer fake-token" }),
      }),
    );
    expect(result).toEqual(project);
  });
```

- [ ] **Step 2: Confirm it fails**

```bash
cd apps/web
pnpm test
```

Expected: fetchProject is not exported.

- [ ] **Step 3: Implement `fetchProject`**

Append to `apps/web/src/app/(app)/api-client.ts`:

```ts
export function fetchProject(token: string, id: string): Promise<Project> {
  return request<Project>(`/projects/${id}`, { method: "GET" }, token);
}
```

- [ ] **Step 4: Run tests and confirm they pass**

```bash
pnpm test
```

Expected: 5 tests pass (4 previous + 1 new).

- [ ] **Step 5: Implement the detail page**

Create `apps/web/src/app/(app)/projects/[id]/page.tsx`:

```tsx
import { auth } from "@clerk/nextjs/server";

import { fetchProject } from "../../api-client";

export default async function ProjectDetailPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const { getToken } = await auth();
  const token = await getToken();
  if (!token) throw new Error("Expected auth");

  const project = await fetchProject(token, id);

  return (
    <div>
      <h1 className="text-2xl font-bold">{project.name}</h1>
      <p className="text-gray-600">{project.category}</p>

      <section className="mt-8 rounded-md border border-dashed p-6 text-gray-500">
        <h2 className="mb-2 text-lg font-semibold text-gray-800">Data sources</h2>
        <p>
          Upload your five files here. This area is wired up in Plan 1 — for now it's a
          placeholder.
        </p>
      </section>
    </div>
  );
}
```

- [ ] **Step 6: Smoke test in the browser**

Sign in, create a project, get redirected to /projects/<id>. Expected: see project name + placeholder.

- [ ] **Step 7: Commit**

```bash
cd ../..
git add apps/web/src/app/\(app\)/projects/\[id\]/page.tsx apps/web/src/app/\(app\)/api-client.ts apps/web/__tests__/api-client.test.ts
git commit -m "feat(web): add project detail page skeleton"
```

---

## Task 21: GitHub Actions CI [scaffold]

**Files:**
- Create: `.github/workflows/ci.yml`

- [ ] **Step 1: Write the CI workflow**

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  api:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: appuser
          POSTGRES_PASSWORD: localdev
          POSTGRES_DB: store_clustering
        options: >-
          --health-cmd "pg_isready -U appuser -d store_clustering"
          --health-interval 5s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 5s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install uv
        run: pip install uv

      - name: Install API dependencies
        working-directory: apps/api
        run: uv sync --all-extras

      - name: Lint API
        working-directory: apps/api
        run: uv run ruff check .

      - name: Run migrations
        working-directory: apps/api
        env:
          DATABASE_URL: postgresql+psycopg://appuser:localdev@localhost:5432/store_clustering
          REDIS_URL: redis://localhost:6379/0
          CLERK_SECRET_KEY: sk_test_ci
          R2_ACCOUNT_ID: ci
          R2_ACCESS_KEY_ID: ci
          R2_SECRET_ACCESS_KEY: ci
          R2_BUCKET: ci
          APP_ENV: test
        run: uv run alembic upgrade head

      - name: Run API tests
        working-directory: apps/api
        env:
          DATABASE_URL: postgresql+psycopg://appuser:localdev@localhost:5432/store_clustering
          REDIS_URL: redis://localhost:6379/0
          CLERK_SECRET_KEY: sk_test_ci
          R2_ACCOUNT_ID: ci
          R2_ACCESS_KEY_ID: ci
          R2_SECRET_ACCESS_KEY: ci
          R2_BUCKET: ci
          APP_ENV: test
        run: uv run pytest -v

  worker:
    runs-on: ubuntu-latest
    needs: api
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Install uv
        run: pip install uv
      - name: Install worker dependencies
        working-directory: apps/worker
        run: uv sync --all-extras
      - name: Run worker tests
        working-directory: apps/worker
        env:
          DATABASE_URL: postgresql+psycopg://appuser:localdev@localhost:5432/store_clustering
          REDIS_URL: redis://localhost:6379/0
          CLERK_SECRET_KEY: sk_test_ci
          R2_ACCOUNT_ID: ci
          R2_ACCESS_KEY_ID: ci
          R2_SECRET_ACCESS_KEY: ci
          R2_BUCKET: ci
        run: uv run pytest -v

  web:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "pnpm"
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Lint web
        working-directory: apps/web
        run: pnpm lint
      - name: Test web
        working-directory: apps/web
        env:
          NEXT_PUBLIC_API_BASE_URL: http://test-api
          NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: pk_test_ci
          CLERK_SECRET_KEY: sk_test_ci
        run: pnpm test
      - name: Build web
        working-directory: apps/web
        env:
          NEXT_PUBLIC_API_BASE_URL: http://test-api
          NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: pk_test_ci
          CLERK_SECRET_KEY: sk_test_ci
        run: pnpm build
```

- [ ] **Step 2: Push to a feature branch and open a PR**

```bash
git checkout -b ci/initial
git add .github/workflows/ci.yml
git commit -m "ci: add initial CI pipeline for api, worker, web"
git push -u origin ci/initial
# Open PR on GitHub
```

Wait for CI to run. Fix any failures inline (most likely small env/path issues). When green, merge the PR.

- [ ] **Step 3: After merge, verify on main**

```bash
git checkout main
git pull
```

CI on main should also pass.

---

## Task 22: Dockerfiles for api, worker, web

**Files:**
- Create: `apps/api/Dockerfile`
- Create: `apps/worker/Dockerfile`
- Create: `apps/web/Dockerfile`

- [ ] **Step 1: API Dockerfile**

Create `apps/api/Dockerfile`:

```dockerfile
FROM python:3.12-slim AS base

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    UV_PROJECT_ENVIRONMENT=/app/.venv \
    PATH="/app/.venv/bin:$PATH"

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 ca-certificates && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir uv

COPY apps/api/pyproject.toml apps/api/uv.lock ./apps/api/
RUN cd apps/api && uv sync --frozen --no-dev

COPY apps/api/src ./apps/api/src
COPY apps/api/alembic ./apps/api/alembic
COPY apps/api/alembic.ini ./apps/api/

WORKDIR /app/apps/api

EXPOSE 8000
CMD ["sh", "-c", "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000"]
```

- [ ] **Step 2: Worker Dockerfile**

Create `apps/worker/Dockerfile`:

```dockerfile
FROM python:3.12-slim AS base

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    UV_PROJECT_ENVIRONMENT=/app/.venv \
    PATH="/app/.venv/bin:$PATH"

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 ca-certificates && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir uv

# Worker depends on the api package via path, so we copy api source too.
COPY apps/api ./apps/api
COPY apps/worker/pyproject.toml apps/worker/uv.lock ./apps/worker/
RUN cd apps/worker && uv sync --frozen --no-dev

COPY apps/worker/src ./apps/worker/src

WORKDIR /app/apps/worker

CMD ["dramatiq", "worker.main", "--processes", "1", "--threads", "4"]
```

- [ ] **Step 3: Web Dockerfile**

Create `apps/web/Dockerfile`:

```dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
RUN corepack enable && corepack prepare pnpm@9.12.0 --activate
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/web/package.json ./apps/web/
RUN pnpm install --frozen-lockfile

FROM node:20-alpine AS builder
WORKDIR /app
RUN corepack enable && corepack prepare pnpm@9.12.0 --activate
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/apps/web/node_modules ./apps/web/node_modules
COPY . .
WORKDIR /app/apps/web
RUN pnpm build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/apps/web/.next ./.next
COPY --from=builder /app/apps/web/public ./public
COPY --from=builder /app/apps/web/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node_modules/.bin/next", "start"]
```

- [ ] **Step 4: Smoke-build each Dockerfile**

From the repo root:

```bash
docker build -f apps/api/Dockerfile -t store-clustering-api:smoke .
docker build -f apps/worker/Dockerfile -t store-clustering-worker:smoke .
docker build -f apps/web/Dockerfile -t store-clustering-web:smoke .
```

Each should build without error.

- [ ] **Step 5: Commit**

```bash
git add apps/api/Dockerfile apps/worker/Dockerfile apps/web/Dockerfile
git commit -m "chore: add Dockerfiles for api, worker, web"
```

---

## Task 23: Fly.io deployment manifests + Neon + Upstash setup [scaffold]

**Files:**
- Create: `apps/api/fly.toml`
- Create: `apps/worker/fly.toml`
- Create: `apps/web/fly.toml`
- Create: `docs/deployment.md`

- [ ] **Step 1: Provision external services**

In order:

1. **Neon Postgres** — create a project; copy the connection string. Use the `psycopg` driver, so the URL should be `postgresql+psycopg://...` (Neon gives you `postgresql://...`; replace the scheme).
2. **Upstash Redis** — create a database; copy the Redis URL.
3. **Cloudflare R2** — confirm bucket exists; copy account ID + access keys.
4. **Clerk** — in the production instance, copy publishable + secret keys.

- [ ] **Step 2: Create three Fly apps**

```bash
fly auth login

fly apps create store-clustering-api
fly apps create store-clustering-worker
fly apps create store-clustering-web
```

- [ ] **Step 3: Set Fly secrets per app**

```bash
fly secrets set -a store-clustering-api \
  DATABASE_URL="postgresql+psycopg://..." \
  REDIS_URL="rediss://..." \
  CLERK_SECRET_KEY="sk_live_..." \
  R2_ACCOUNT_ID="..." \
  R2_ACCESS_KEY_ID="..." \
  R2_SECRET_ACCESS_KEY="..." \
  R2_BUCKET="store-clustering-prod" \
  APP_ENV="production"

fly secrets set -a store-clustering-worker \
  DATABASE_URL="postgresql+psycopg://..." \
  REDIS_URL="rediss://..." \
  CLERK_SECRET_KEY="sk_live_..." \
  R2_ACCOUNT_ID="..." \
  R2_ACCESS_KEY_ID="..." \
  R2_SECRET_ACCESS_KEY="..." \
  R2_BUCKET="store-clustering-prod" \
  APP_ENV="production"

fly secrets set -a store-clustering-web \
  NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="pk_live_..." \
  CLERK_SECRET_KEY="sk_live_..." \
  NEXT_PUBLIC_API_BASE_URL="https://store-clustering-api.fly.dev"
```

- [ ] **Step 4: Write `apps/api/fly.toml`**

```toml
app = "store-clustering-api"
primary_region = "iad"

[build]
dockerfile = "Dockerfile"

[env]
LOG_LEVEL = "INFO"

[http_service]
internal_port = 8000
force_https = true
auto_stop_machines = "stop"
auto_start_machines = true
min_machines_running = 1

[[http_service.checks]]
  grace_period = "10s"
  interval = "30s"
  method = "GET"
  path = "/health"
  protocol = "http"
  timeout = "5s"

[[vm]]
cpu_kind = "shared"
cpus = 1
memory_mb = 512
```

- [ ] **Step 5: Write `apps/worker/fly.toml`**

```toml
app = "store-clustering-worker"
primary_region = "iad"

[build]
dockerfile = "Dockerfile"

[env]
LOG_LEVEL = "INFO"

[[vm]]
cpu_kind = "shared"
cpus = 1
memory_mb = 1024  # workers may need more for clustering later
```

- [ ] **Step 6: Write `apps/web/fly.toml`**

```toml
app = "store-clustering-web"
primary_region = "iad"

[build]
dockerfile = "Dockerfile"

[http_service]
internal_port = 3000
force_https = true
auto_stop_machines = "stop"
auto_start_machines = true
min_machines_running = 1

[[vm]]
cpu_kind = "shared"
cpus = 1
memory_mb = 512
```

- [ ] **Step 7: Deploy each app (from repo root, using the root context for Docker build)**

```bash
fly deploy -c apps/api/fly.toml --dockerfile apps/api/Dockerfile
fly deploy -c apps/worker/fly.toml --dockerfile apps/worker/Dockerfile
fly deploy -c apps/web/fly.toml --dockerfile apps/web/Dockerfile
```

- [ ] **Step 8: Smoke-test prod**

```bash
curl https://store-clustering-api.fly.dev/health
# Expected: {"status":"ok"}

open https://store-clustering-web.fly.dev
# Expected: marketing page; sign in / sign up via Clerk; redirect to projects list; create a project; see it.
```

- [ ] **Step 9: Write `docs/deployment.md`**

Create `docs/deployment.md`:

```markdown
# Deployment

Three Fly apps. Provisioned externals: Neon Postgres, Upstash Redis,
Cloudflare R2, Clerk (production instance).

## Apps

| Name | Purpose | URL |
|---|---|---|
| `store-clustering-api` | FastAPI backend | https://store-clustering-api.fly.dev |
| `store-clustering-worker` | Dramatiq worker | (no public URL) |
| `store-clustering-web` | Next.js frontend | https://store-clustering-web.fly.dev |

## Deploy

From the repo root:

\`\`\`bash
fly deploy -c apps/api/fly.toml --dockerfile apps/api/Dockerfile
fly deploy -c apps/worker/fly.toml --dockerfile apps/worker/Dockerfile
fly deploy -c apps/web/fly.toml --dockerfile apps/web/Dockerfile
\`\`\`

## Secrets

See `fly secrets list -a <app>` per app. Every secret listed in `.env.example`
must be set in Fly.

## Migrations

Run automatically on API container start via the Dockerfile CMD
(`alembic upgrade head`).

## Health

| Check | Command | Expected |
|---|---|---|
| API health | `curl https://store-clustering-api.fly.dev/health` | `{"status":"ok"}` |
| Web is up | `curl -I https://store-clustering-web.fly.dev` | 200 |
| Worker is consuming | `fly logs -a store-clustering-worker` | "Booting worker" |
```

- [ ] **Step 10: Commit**

```bash
git add apps/api/fly.toml apps/worker/fly.toml apps/web/fly.toml docs/deployment.md
git commit -m "chore: add Fly.io deployment manifests and docs"
```

---

## Task 24: Deployment workflow [scaffold]

**Files:**
- Create: `.github/workflows/deploy.yml`

- [ ] **Step 1: Write the deploy workflow**

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --remote-only -c apps/api/fly.toml --dockerfile apps/api/Dockerfile
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}

  deploy-worker:
    runs-on: ubuntu-latest
    needs: deploy-api
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --remote-only -c apps/worker/fly.toml --dockerfile apps/worker/Dockerfile
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}

  deploy-web:
    runs-on: ubuntu-latest
    needs: deploy-api
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --remote-only -c apps/web/fly.toml --dockerfile apps/web/Dockerfile
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

- [ ] **Step 2: Add `FLY_API_TOKEN` to GitHub repo secrets**

```bash
fly auth token
# Copy the token, then in GitHub: Settings → Secrets → New repository secret
# Name: FLY_API_TOKEN, value: <token>
```

- [ ] **Step 3: Commit and push**

```bash
git add .github/workflows/deploy.yml
git commit -m "chore: add deploy workflow for main branch"
git push origin main
```

- [ ] **Step 4: Verify the deploy workflow runs on the push**

Watch GitHub Actions tab. All three deploys should succeed.

- [ ] **Step 5: Smoke-test prod once more**

```bash
curl https://store-clustering-api.fly.dev/health
```

Expected: still `{"status":"ok"}`.

---

## Task 25: End-of-plan smoke test (manual)

This isn't a code task — it's the acceptance check for Plan 0.

- [ ] **Step 1: As a brand-new user, do the entire end-to-end flow**

In a private browser window:

1. Go to https://store-clustering-web.fly.dev.
2. Click sign-up. Sign up with a fresh email.
3. Create or join an organization (Clerk's flow).
4. Land on `/projects`. Expected: "No projects yet" empty state.
5. Click "New project". Fill in name = "Plan 0 Smoke Test", category = "Frozen Pizza".
6. Submit. Expected: redirected to `/projects/<id>`.
7. See project name, category, and "Data sources" placeholder.
8. Navigate back to `/projects`. Expected: see the new project in the list.

- [ ] **Step 2: Verify multi-tenancy from the database side**

```bash
# In a separate Neon SQL editor session
SELECT id, organization_id, name, category FROM projects ORDER BY created_at DESC LIMIT 5;
SELECT id, clerk_org_id, name FROM organizations ORDER BY created_at DESC LIMIT 5;
SELECT id, clerk_user_id, organization_id, email FROM users ORDER BY created_at DESC LIMIT 5;
```

Expected: see your new org, user, and project. `projects.organization_id` should match `users.organization_id`.

- [ ] **Step 3: Tag the release**

```bash
git tag v0.0.1
git push origin v0.0.1
```

Plan 0 is complete.

---

## Self-review

A quick check against the spec before we hand off to the next plan.

**Spec coverage (Plan 0's slice only — the rest is later plans):**

- §3 multi-tenancy: ✅ enforced via Clerk org claims → `org_id` filtering in `_load_owned_project` and `list_projects`. Cross-cutting isolation test in Task 13.
- §5 architecture: ✅ all six components scaffolded — browser (web), API (api), DB (Postgres via Neon), object storage (R2 client, signed-URL endpoint), compute worker (Dramatiq ping actor), auth provider (Clerk).
- §4 domain model: Plan 0 implements Organization, User, Project. RefinementDrafts, ClusteringRun, DataSources, Scenarios, etc. are out of scope — they come in Plans 1–5.
- §7.5 provider abstraction: not yet — implemented in Plan 1 (when there's actually data to ingest). Plan 0 only puts in the R2 signed-URL plumbing.

**Placeholder scan:** No `TBD`, no `TODO`, no "implement later" in steps. The LICENSE file says "License terms TBD" — that's intentional, marking real legal-team work to be done separately.

**Type consistency:**
- `Project.id` is `UUID` in SQLAlchemy and `str` in API responses (intentional — JSON serialization).
- `Project` Pydantic schema in `projects.py` matches `Project` TS type in `api-client.ts` (id/name/category — three fields).
- `FileKind` enum in `uploads.py` matches the five canonical file types from spec §7.1.

**Things deliberately deferred to later plans:**
- File upload UI and column mapping (Plan 1).
- The five DataSource SQLAlchemy models (Plan 1).
- Validation rules + orphan store error (Plan 1).
- ClusteringRun model + scenario canvas (Plan 2).
- All other domain entities and endpoints.

---

## Execution Handoff

**Plan complete and saved to `docs/superpowers/plans/2026-06-01-store-clustering-plan-0-scaffolding.md`. Two execution options:**

**1. Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration. Best for a plan this long.

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints. Slower turns, more linear.

**Which approach?**

(Or — if you want to pause here and read the plan first before executing anything, just say so. We can also commit the plan to git before kicking off implementation.)
