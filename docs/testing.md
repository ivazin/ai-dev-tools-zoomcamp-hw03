# Testing Guide

This document describes how to execute tests across different environments for **Agent Relay**.

---

## 1. Test Suite Overview

- **Unit / Protocol Tests (`tests/unit/test_agent_relay.py`)**:
  - Tests basic agent registration, task submission, authentication boundaries, claim tokens, and idempotency.
  - Tests SQLite WAL concurrency guarantees and recovery loop.
  - Defaults to an isolated scratch SQLite database at `/tmp/agent-relay-test.db`.

- **Integration Tests (`tests/integration/test_postgres.py`)**:
  - Validates multi-worker concurrency using PostgreSQL with row-level locks (`SELECT ... FOR UPDATE SKIP LOCKED`).
  - Ensures tasks are claimed without duplicate delivery across parallel worker threads.
  - Requires `RELAY_DATABASE_URL` pointing to a running PostgreSQL instance.

---

## 2. Running Tests Locally

### Quick Run (SQLite Default)

```bash
uv run pytest -v tests/unit/
```

### Running Integration Tests with PostgreSQL

1. **Start PostgreSQL via Docker / Docker Compose**:
   ```bash
   docker compose up -d postgres
   ```

2. **Run tests against PostgreSQL**:
   ```bash
   RELAY_DATABASE_URL="postgresql+psycopg://relay:relay_password@localhost:5432/agent_relay" uv run pytest -v
   ```

3. **Run only the integration test suite**:
   ```bash
   RELAY_DATABASE_URL="postgresql+psycopg://relay:relay_password@localhost:5432/agent_relay" uv run pytest -v tests/integration/
   ```

---

## 3. Running in CI / Locally with `act`

The CI pipeline runs automated checks via GitHub Actions (`.github/workflows/ci.yml`).

To run the full CI workflow locally with `act`:

```bash
act -j test-build-deploy
```

This starts a clean PostgreSQL service container, runs both unit and integration tests, builds the Docker container, and verifies the deployment.
