# Week 3 Project — CI/CD + Live Deploy

## What you submit

**One public GitHub repository** forked from the Week 3 starter, with Docker, CI, and CD added by you.

Submit on **[csot-devops.devclub.in/submission](https://csot-devops.devclub.in/submission)** → Week 3.

| Field | Value |
|-------|-------|
| GitHub repository URL | your fork URL |
| Assignment folder | `.` |
| Fine-grained PAT | **Required** — Contents read+write + Actions read |

---

## Starter (fork this)

**[github.com/sumit-IITD/csot-w3-starter](https://github.com/sumit-IITD/csot-w3-starter)**

The starter has:
- `app/main.py` — Flask app with `GET /health` and `GET /api/version`
- `VERSION` file at repo root (contains `1.0.0`)
- **No** Dockerfile, **no** compose, **no** CI, **no** tests

You add everything.

---

## Act 1 — CI & containerize (200 pts)

100 raw points scaled to 200:

| Section | Checks | Pts |
|---------|--------|-----|
| **Test quality** | tests/ files, CI runs pytest, coverage ≥70%, submitted commit CI green, AI test review | 40 |
| **Linting & quality** | lint job, linter tool, secret scan, dependency scan | 20 |
| **Docker quality** | Dockerfile, compose.yaml, port `"0:80"`, SQLite volume, build-push in CI | 25 |
| **Structure & deploy** | VERSION file, `/api/version` in code, workflows dir, deploy job with SSH | 15 |

### Required endpoints

| Endpoint | Response |
|----------|----------|
| `GET /health` | `{"status": "ok"}` |
| `GET /api/version` | `{"version": "<VERSION file content>"}` |

---

## Act 2 — Live CD (100 pts)

After Act 1 grades, go to the portal submission page:

| Step | Pts |
|------|-----|
| Workflow has a deploy job (SSH + `docker compose pull/up`) | 10 |
| Click **Verify live 1.0.0** — grader pings your app URL | 45 |
| Grader commits `VERSION=1.0.1` → your CI builds + deploys → live shows `1.0.1` | 45 |

### Deploy sandbox secrets (from portal)

| GitHub Secret | Value |
|---------------|-------|
| `DEPLOY_HOST` | sandbox IP |
| `DEPLOY_SSH_PORT` | SSH port |
| `DEPLOY_USER` | `incident` |
| `DEPLOY_PASSWORD` | sandbox password |
| `DEPLOY_PATH` | deploy directory |
| `COMPOSE_PROJECT_NAME` | compose project |

---

## Total: 300 pts
