# Release Process

This document outlines the versioning, tagging, and automated release lifecycle for **Agent Relay**.

---

## 1. Versioning Strategy

Agent Relay follows [Semantic Versioning (SemVer)](https://semver.org/):
- **MAJOR (`X.0.0`)**: Incompatible API breaking changes or protocol modifications.
- **MINOR (`0.X.0`)**: Backward-compatible features (new routes, extra endpoints, storage adapters).
- **PATCH (`0.0.X`)**: Bug fixes and minor performance optimizations.

---

## 2. CI/CD Lifecycle

```text
[ Feature Branch / PR ]
         │
         ▼
[ CI Workflow: .github/workflows/ci.yml ]
  ├── Run Unit Tests (tests/unit/)
  ├── Run PostgreSQL Integration Tests (tests/integration/)
  ├── Build Docker Image
  └── Test Local Rollout in kind
         │
         ▼ (Merged to main)
[ Deploy Workflow: .github/workflows/deploy.yml ]
  └── Apply manifests & update container images
```

---

## 3. Creating a Release

1. **Update Project Version**:
   Update `version` in `pyproject.toml` and `main.py` (FastAPI title/version).

2. **Commit and Tag**:
   ```bash
   git add pyproject.toml main.py
   git commit -m "chore(release): bump version to v0.2.0"
   git tag -a v0.2.0 -m "Release v0.2.0"
   git push origin main --tags
   ```

3. **Automated Deployment**:
   - The CI pipeline executes tests against the commit.
   - Images are published and tagged with the semantic tag as well as the Git commit SHA.
   - The deployment workflow rolls out the updated container version without downtime.
