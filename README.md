# Indico Fullstack Developer

Fullstack application consisting of a **Go REST API backend** and a **Next.js frontend**, built as part of the Indico Developer Assignment.

This repository uses **git submodules** — `indico-be` and `indico-fe` are separate repositories.

## Clone

```bash
# Clone with submodules in one step
git clone --recurse-submodules https://github.com/shoelfikar/indico-fullstack-project.git

# Or if already cloned without submodules
git submodule update --init --recursive
```

### Submodule Repositories

| Submodule | Repository |
|-----------|------------|
| `indico-be/` | [shoelfikar/indico-backend](https://github.com/shoelfikar/indico-backend) |
| `indico-fe/` | [shoelfikar/indico-frontend](https://github.com/shoelfikar/indico-frontend) |

## Projects

| Service | Directory | Tech | Port |
|---------|-----------|------|------|
| Backend API | `indico-be/` | Go 1.25, Gin, PostgreSQL, pgx/v5 | `:8080` |
| Frontend UI | `indico-fe/` | Next.js 16, TypeScript, Material UI v7 | `:3000` |
| Database | — | PostgreSQL 16 | `:5432` |

## Backend — Order & Settlement Service

A Go REST API with two core features:

**1. Order Management** — Concurrency-safe product ordering with limited stock. Uses atomic `UPDATE ... WHERE stock >= qty` to guarantee no overselling even under 500 concurrent requests.

**2. Settlement Background Job** — Processes ~1M transactions into aggregated settlements per merchant per day. Uses a channel-based worker pool with batch-level fan-out for parallel processing. Outputs CSV files downloadable via API.

### API Endpoints

```
GET    /health                     Health check
POST   /api/v1/orders/             Create order (atomic stock decrement)
GET    /api/v1/orders/:id          Get order by ID
POST   /api/v1/jobs/settlement     Enqueue settlement job (async, 202)
GET    /api/v1/jobs/:id            Poll job status & progress
POST   /api/v1/jobs/:id/cancel     Cancel a running job
GET    /api/v1/downloads/:id       Download settlement CSV
```

See [indico-be/README.md](indico-be/README.md) for full API documentation, curl examples, and architecture details.

## Frontend — User Management UI

A single-page application for managing users, built with Next.js App Router and Material UI.

### Features

- List, search (debounced), and paginate users
- Create, edit, and delete users with form validation (React Hook Form + Zod)
- Dark theme, loading states, error handling, toast notifications
- Server state management via TanStack Query v5

See [indico-fe/README.md](indico-fe/README.md) for full feature list and project structure.

## Quick Start

### Prerequisites

- Docker & Docker Compose

### Setup Environment

Create a `.env` file in the root directory for Docker Compose:

```bash
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=indico_db
DB_SSLMODE=disable
```

### Run All Services

```bash
docker compose up --build -d
```

This starts:
- **PostgreSQL** on port `5432` (with healthcheck)
- **Backend** on port `8080` (auto-runs migrations and seeds 1M transactions on first start)
- **Frontend** on port `3000`

### Verify

```bash
# Backend health check
curl http://localhost:8080/health

# Frontend
open http://localhost:3000
```

### Stop

```bash
docker compose down
```

### Reset Database

```bash
docker compose down -v   # removes volumes (postgres data)
docker compose up --build -d
```

## Environment Variables

Create a `.env` file in the root directory to override defaults:

```env
# Database
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=indico_db
DB_PORT=5432
DB_SSLMODE=disable

# Backend
PORT=8080
WORKERS=8
BATCH_SIZE=10000
RESULT_DIR=/tmp/settlements

# Frontend
NEXT_PUBLIC_API_URL=https://jsonplaceholder.typicode.com
```

## Testing

### Backend Unit Tests

```bash
cd indico-be
make test
```

### Backend Integration Test — 500 Concurrent Orders

Proves no overselling: 500 goroutines concurrently order from a product with stock=100.

```bash
cd indico-be

# Ensure server + DB are running, then:
make test-integration
```

See [indico-be/README.md](indico-be/README.md) for details.

## Project Structure

```
indico-fullstack-developer/
├── docker-compose.yml          # All services (db + backend + frontend)
├── indico-be/                  # Go backend
│   ├── cmd/server/             # Entry point
│   ├── internal/               # Handlers, services, repositories, workers
│   ├── database/migrations/    # SQL migrations (auto-run on startup)
│   ├── tests/integration/      # HTTP integration tests
│   ├── docker-compose.yml      # Backend-only compose (with its own DB)
│   └── Dockerfile
├── indico-fe/                  # Next.js frontend
│   ├── src/                    # App Router, components, hooks
│   └── Dockerfile
└── test-concurrent-orders.mjs  # Standalone JS concurrent order test
```
