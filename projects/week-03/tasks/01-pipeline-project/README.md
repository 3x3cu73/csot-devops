# Week 3 Project — CI/CD + Live Deploy

## Quick start

1. Fork [**csot-w3-starter**](https://github.com/sumit-IITD/csot-w3-starter) — Flask + SQLite, no Docker/CI yet.
2. Build everything listed below.
3. Submit on the **[submission portal](https://csot-devops.devclub.in/submission)** → Week 3 (paste a fine-grained PAT).
4. Copy your **deploy sandbox** credentials into GitHub secrets.
5. Click **Verify live 1.0.0** once your app is deployed.

---

## Act 1 — CI & containerize (200 pts)

| Check | Pts |
|-------|-----|
| `tests/` with `test_*.py` files | 5 |
| CI workflow runs pytest | 5 |
| Coverage ≥70% gate (`--cov-fail-under=70`) | 5 |
| Submitted commit CI all green | 10 |
| AI test review — route coverage, schema, # of tests | 15 |
| Lint job in CI | 5 |
| Linter tool (ruff / flake8 / pylint) | 5 |
| Secret scan (TruffleHog / gitleaks) | 5 |
| Dependency scan (pip-audit) | 5 |
| `Dockerfile` present | 5 |
| `compose.yaml` present | 5 |
| Web service uses `ports: ["0:80"]` | 5 |
| SQLite volume / bind-mount in compose | 5 |
| `docker/build-push-action` in CI (GHCR) | 5 |
| `VERSION` file at repo root | 3 |
| `GET /api/version` endpoint in app code | 3 |
| `.github/workflows/` with CI files | 4 |
| Deploy job with SSH in workflow | 5 |
| **Total (scaled to 200)** | **100 raw** |

---

## Act 2 — Live CD (100 pts)

| Check | Pts |
|-------|-----|
| Deploy job exists in workflow (SSH + `docker compose`) | 10 |
| Live app serves version `1.0.0` (click **Verify live 1.0.0** on portal) | 45 |
| Grader commits `VERSION=1.0.1` → your CI/CD redeploys → live shows `1.0.1` | 45 |

### Required endpoints

| Endpoint | Response |
|----------|----------|
| `GET /health` | `{"status": "ok"}` |
| `GET /api/version` | `{"version": "<VERSION file content>"}` |

### Deploy sandbox secrets

From the portal → Settings → GitHub Secrets:

| Secret | Value |
|--------|-------|
| `DEPLOY_HOST` | sandbox IP |
| `DEPLOY_SSH_PORT` | SSH port |
| `DEPLOY_USER` | `incident` |
| `DEPLOY_PASSWORD` | sandbox password |
| `DEPLOY_PATH` | deploy directory |
| `COMPOSE_PROJECT_NAME` | compose project |

---

## Points: **300**
