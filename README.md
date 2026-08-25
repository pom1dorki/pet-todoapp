# Golang Todo API

Production-style REST API for a todo application written in pure Go (`net/http`).

**Live demo:** [http://5.101.50.176:5050](http://5.101.50.176:5050)  
**Swagger UI:** available at `/swagger/index.html` when the service is running.

---

## Features

- **Users** — full CRUD with optimistic locking (`version` field)
- **Tasks** — full CRUD, linked to author, completion tracking, optimistic locking
- **Statistics** — created / completed tasks count, completion rate, average completion time
- **Pagination & filtering** for list endpoints
- **Request validation** via `go-playground/validator`
- **Structured logging** with `zap`
- **Middleware**: CORS, Request-ID, logging, tracing, panic recovery
- **API versioning** (`/api/v1`)
- **Swagger / OpenAPI** documentation (auto-generated)
- **Database migrations** (golang-migrate)
- **Docker Compose** ready (app + PostgreSQL)
- **Graceful shutdown**
- Simple web UI served from `/` (dark/light theme)

---

## Tech Stack

| Layer            | Technology                          |
|------------------|-------------------------------------|
| Language         | Go 1.26+                            |
| HTTP             | Standard library `net/http`         |
| Database         | PostgreSQL 18 + `jackc/pgx/v5`      |
| Migrations       | `golang-migrate`                    |
| Validation       | `go-playground/validator`           |
| Logging          | `uber-go/zap`                       |
| Config           | Environment variables (`envconfig`) |
| Documentation    | `swaggo/swag`                       |
| Containerization | Docker + Docker Compose             |

---

## Architecture

Clean layered architecture:

```
cmd/todoapp/          → entrypoint
internal/
  core/               → shared infrastructure
    config/
    domain/           → pure domain models + validation
    errors/
    logger/
    repository/       → connection pool
    transport/http/   → server, middleware, request/response helpers
  features/
    users/
    tasks/
    statistics/
    web/              → static frontend
migrations/
docs/                 → generated Swagger
public/               → frontend assets
```

Each feature follows the classic flow:

```
HTTP Handler → Service → Repository (PostgreSQL)
```

Domain models live in `internal/core/domain` and contain validation logic and optimistic-locking version field.

---

## Quick Start (Docker)

### 1. Clone & configure

```bash
git clone https://github.com/pom1dorki/pet-todoapp.git
cd pet-todoapp
cp .env.example .env
```

Edit `.env` and set at least:

```env
POSTGRES_USER=todo
POSTGRES_PASSWORD=secret
POSTGRES_DB=todoapp
```

### 2. Start PostgreSQL + run migrations

```bash
make env-up          # start postgres
make migrate-up      # apply migrations
```

### 3. Run the application

**Option A — local Go binary (recommended for development):**

```bash
make todoapp-run
```

**Option B — full Docker deployment:**

```bash
make todoapp-deploy
```

Service will be available at **http://localhost:5050**

- Web UI: http://localhost:5050  
- Swagger: http://localhost:5050/swagger/index.html  
- API base: http://localhost:5050/api/v1

---

## Makefile commands

| Command              | Description                              |
|----------------------|------------------------------------------|
| `make env-up`        | Start PostgreSQL container               |
| `make env-down`      | Stop PostgreSQL                          |
| `make migrate-up`    | Apply all migrations                     |
| `make migrate-down`  | Rollback last migration                  |
| `make todoapp-run`   | Run app locally (`go run`)               |
| `make todoapp-deploy`| Build & start app container              |
| `make todoapp-undeploy` | Stop app container                    |
| `make swagger-gen`   | Regenerate Swagger docs                  |
| `make ps`            | Show running containers                  |

---

## API Overview

Base path: `/api/v1`

### Users
- `POST   /users` — create user
- `GET    /users` — list users (pagination)
- `GET    /users/{id}` — get user
- `PATCH  /users/{id}` — partial update (optimistic locking)
- `DELETE /users/{id}` — delete user

### Tasks
- `POST   /tasks` — create task
- `GET    /tasks` — list tasks (pagination + filters)
- `GET    /tasks/{id}` — get task
- `PATCH  /tasks/{id}` — partial update (optimistic locking)
- `DELETE /tasks/{id}` — delete task

### Statistics
- `GET    /statistics` — aggregated task statistics

Full interactive documentation is available via **Swagger UI**.

---

## Optimistic Locking

Both `users` and `tasks` tables have a `version` column.

On every update the client must send the current `version`.  
If the version in the database has already changed, the API returns a conflict error.  
This protects against lost updates in concurrent scenarios.

---

## Project Structure (simplified)

```
.
├── cmd/todoapp/
│   ├── main.go
│   └── Dockerfile
├── internal/
│   ├── core/                 # shared code
│   └── features/
│       ├── users/
│       ├── tasks/
│       ├── statistics/
│       └── web/
├── migrations/
├── docs/                     # swagger artifacts
├── public/                   # frontend
├── docker-compose.yaml
├── Makefile
├── .env.example
└── go.mod
```

---

## Configuration

All configuration is done through environment variables (see `.env.example`):

| Variable                 | Description                        | Default     |
|--------------------------|------------------------------------|-------------|
| `HTTP_ADDR`              | Listen address                     | `:5050`     |
| `HTTP_SHUTDOWN_TIMEOUT`  | Graceful shutdown timeout          | `30s`       |
| `ALLOWED_ORIGINS`        | CORS allowed origins               | —           |
| `POSTGRES_HOST`          | PostgreSQL host                    | —           |
| `POSTGRES_USER`          | PostgreSQL user                    | —           |
| `POSTGRES_PASSWORD`      | PostgreSQL password                | —           |
| `POSTGRES_DB`            | Database name                      | —           |
| `POSTGRES_TIMEOUT`       | Connection timeout                 | `10s`       |
| `LOGGER_LEVEL`           | Log level (`DEBUG`, `INFO`, …)     | `DEBUG`     |
| `TIME_ZONE`              | Application timezone               | `UTC`       |

---

## Development notes

- The project intentionally uses the standard library HTTP server instead of a framework to demonstrate solid understanding of the Go net/http package and middleware patterns.
- Domain validation lives inside domain models.
- Repositories use `pgx` for high-performance PostgreSQL access.
- Migrations are managed with the official `migrate` tool via Docker.

---

## Author

**Ilya Petrishchev**  
GitHub: [pom1dorki](https://github.com/pom1dorki)

---

## License

This is a pet / portfolio project. Feel free to use the code for learning purposes.
