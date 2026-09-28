# Deployment Guide

This guide covers deployment options for **Agent Relay**: Docker Compose (single host) and Kubernetes (via `kind` or production clusters).

---

## 1. Docker Compose Deployment

The stack includes the `agent-relay` application and a `postgres` database service.

### Files
- `docker-compose.yml` (and `compose.yaml`)
- `Dockerfile`

### Starting the Stack
```bash
docker compose up --build -d
```

### Stopping the Stack
```bash
docker compose down
```

### Verification
- **Liveness probe**: `curl http://localhost:8000/health`
- **Readiness probe**: `curl http://localhost:8000/ready`
- **Dashboard UI**: [http://localhost:8000/](http://localhost:8000/)

---

## 2. Kubernetes Deployment (`kind` & Production)

All Kubernetes resource definitions reside in the `k8s/` directory:
- `k8s/postgres.yaml`: PersistentVolumeClaim, Deployment (1 replica), and ClusterIP Service.
- `k8s/agent-relay.yaml`: Multi-replica Deployment (e.g. 4 replicas), liveness/readiness probes, and NodePort Service.

### Prerequisites
- `kind` (`brew install kind`)
- `kubectl`

### Steps

1. **Create the kind cluster**:
   ```bash
   kind create cluster --name agent-relay
   ```

2. **Build and load the Docker image**:
   ```bash
   docker build -t agent-relay:local .
   kind load docker-image agent-relay:local --name agent-relay
   ```

3. **Apply manifests**:
   ```bash
   kubectl apply -f k8s/postgres.yaml
   kubectl rollout status deployment/postgres --timeout=120s

   kubectl apply -f k8s/agent-relay.yaml
   kubectl rollout status deployment/agent-relay --timeout=120s
   ```

4. **Port forward to access locally**:
   ```bash
   kubectl port-forward svc/agent-relay 8000:8000
   ```

5. **Scaling Replicas**:
   ```bash
   kubectl scale deployment/agent-relay --replicas=4
   ```
