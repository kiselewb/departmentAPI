# Organizational Structure API

A REST API for managing the hierarchical structure of departments and employees.

## Tech Stack

- **FastAPI** + **SQLAlchemy 2.0** (async) + **PostgreSQL**
- **Alembic** - migrations
- **Pydantic v2** - data validation
- **loguru** - logging
- **pytest** + **pytest-asyncio** - tests
- **uv** - dependency management
- **Docker** + **docker-compose**

## Project Structure


```
app/
├── api/v1/endpoints/   # Routers (departments, employees)
├── config/             # Configuration, paths, logging
├── database/           # Database session
├── models/             # SQLAlchemy models
├── repositories/       # Database layer
├── schemas/            # Pydantic schemas
├── services/           # Business logic
└── utils/              # Exceptions
alembic/                # Migrations
tests/                  # Integration tests
```

## Running via Docker

### 1. Clone the repository

```bash
git clone <repo-url>
cd <repo-dir>
```

### 2. Create a `.env` file based on the example

```bash
cp .env.example .env
```

> In `.env`, make sure that `DB_HOST=db` (for the Docker network).

### 3. Run

```bash
docker-compose up -d --build
```

The application will start at [http://localhost:8000/api/v1/docs](http://localhost:8000/api/v1/docs).
Migrations are applied automatically before the application starts.

---

## Running locally (without Docker)

### Requirements

- Python 3.13+
- PostgreSQL (running locally)
- uv

### 1. Install dependencies

```bash
uv sync
```

### 2. Configure `.env`

```bash
cp .env.example .env
# Change DB_HOST=localhost
```

### 3. Apply migrations

```bash
uv run alembic upgrade head
```

### 4. Start the server

```bash
uv run fastapi run app/main.py
```

---

## Tests

A separate database is required for testing (specified via `TEST_DB_NAME` in `.env`).

```bash
uv run pytest
```

---

## API

Documentation is available once the application has been launched:

- **Swagger UI**: [http://localhost:8000/api/v1/docs](http://localhost:8000/api/v1/docs)

### Main endpoints

| Method | URL | Description |
|-------|-----|----------|
| `GET` | `/api/v1/departments/` | List of all departments |
| `POST` | `/api/v1/departments/` | Create a department |
| `GET` | `/api/v1/departments/{id}` | Department tree (`depth`, `include_employees`) |
| `PATCH` | `/api/v1/departments/{id}` | Update a department |
| `DELETE` | `/api/v1/departments/{id}` | Delete (`mode=cascade\|reassign`) |
| `POST` | `/api/v1/departments/{id}/employees/` | Add an employee |
| `GET` | `/api/v1/employees/` | List all employees |

### Business logic features

- Department names are unique within a single parent
- A department cannot be its own parent
- When a department is moved, a cycle check is performed (recursive CTE)
- Deletion in `reassign` mode transfers employees to the specified department
- Deletion in `cascade` mode deletes the entire tree along with the employees