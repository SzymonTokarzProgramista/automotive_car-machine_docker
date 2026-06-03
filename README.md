# Automotive Car Machine Docker

Central orchestration and configuration repo for the autonomous vehicle web stack.

This directory is the source of truth for local Docker Compose runtime configuration. Service code lives in sibling repositories:

- `../automotive_car-web_backend`
- `../automotive_car-web_ui`

## Structure

- `.env` - shared runtime environment used by Docker Compose.
- `.env.example` - template for recreating local `.env`.
- `docker-compose.yml` - starts nginx, backend, UI, Postgres, and ThingsBoard CE together.
- `kubernetes/` - Kustomize manifests for running the stack in Kubernetes.
- `settings/` - shared YAML configuration files.
- `certificates/` - local certificates and keys mounted read-only into services.

## Run

```powershell
docker compose up --build
```

Services:

- UI: `http://localhost:5173`
- Nginx proxy: `http://localhost:5173`
- Backend: `http://localhost:8000`
- Postgres: `localhost:5432`
- ThingsBoard CE: `http://localhost:9090`
- ThingsBoard MQTT: `localhost:1883`

The public UI URL is served by nginx. The React dev server runs inside the Compose network as `ui:5173`, while nginx proxies `/` to the UI and `/api`, `/auth`, `/health` to the backend.

## ThingsBoard

ThingsBoard CE is included as a local telemetry dashboard using the `thingsboard/tb-postgres` image. It persists data in Docker volumes:

- `thingsboard_data`
- `thingsboard_logs`

On first startup, open `http://localhost:9090` and use the default ThingsBoard credentials from the official CE image documentation. Vehicle telemetry can be sent directly to the MQTT port or routed later through a dedicated telemetry gateway.

## Kubernetes

The `kubernetes/` directory contains a Kustomize setup for running the same machine stack in a local Kubernetes cluster.

Build local images first:

```powershell
docker build -t automotive-car-backend:local ../automotive_car-web_backend
docker build -t automotive-car-ui:local ../automotive_car-web_ui
```

If you use `kind` or `k3d`, load/import these images into the cluster before applying manifests.

Apply the local overlay:

```powershell
kubectl apply -k kubernetes/overlays/local
```

Local Kubernetes services:

- UI through nginx: `http://localhost:30080`
- ThingsBoard CE: `http://localhost:30090`
- ThingsBoard MQTT NodePort: `localhost:31883`

Useful checks:

```powershell
kubectl -n automotive-car get pods
kubectl -n automotive-car get svc
kubectl -n automotive-car logs deploy/backend
```

The local overlay includes development secrets in `kubernetes/overlays/local/secrets.yaml`. For production, create a separate overlay and replace those values with real Kubernetes Secrets or an external secret manager.

## Configuration Rule

Prefer changing runtime values here instead of editing per-service Compose files. The backend and UI repos can keep standalone Compose files for isolated development, but this repository is the main machine-level configurator.
