# Automotive Car Machine Docker

Central orchestration and configuration repo for the autonomous vehicle web stack.

This directory is the source of truth for local Docker Compose runtime configuration. Service code lives in sibling repositories:

- `../automotive_car-web_backend`
- `../automotive_car-web_ui`

## Structure

- `.env` - shared runtime environment used by Docker Compose.
- `.env.example` - template for recreating local `.env`.
- `docker-compose.yml` - starts backend, UI, and Postgres together.
- `settings/` - shared YAML configuration files.
- `certificates/` - local certificates and keys mounted read-only into services.

## Run

```powershell
docker compose up --build
```

Services:

- UI: `http://localhost:5173`
- Backend: `http://localhost:8000`
- Postgres: `localhost:5432`

## Configuration Rule

Prefer changing runtime values here instead of editing per-service Compose files. The backend and UI repos can keep standalone Compose files for isolated development, but this repository is the main machine-level configurator.
