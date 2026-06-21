# Week 3 Project — Production CI/CD Pipeline

## What you submit

**One public GitHub repository** — the same repo as your Week 2 Docker project, now with CI/CD on top.

Submit on **[csot-devops.devclub.in/submission](https://csot-devops.devclub.in/submission)** → Week 3:

| Field | Example |
|-------|---------|
| GitHub repository URL | `https://github.com/you/my-app` |
| Assignment folder | `.` (repo root) |

After you submit, the autograder **clones your repo** and scores it within a few minutes. Check the **leaderboard** and your dashboard for the breakdown.

---

## Prerequisites (from Week 2)

Your repo must already contain:

- `Dockerfile` — builds your app
- `compose.yaml` (or `docker-compose.yml`) — **≥2 services** (app + database), with a **healthcheck**
- App that runs with `docker compose up --build`

Week 3 adds `.github/workflows/` — you are not re-proving Docker from scratch, but the grader **does** run `docker build` and `docker compose config` on your submission.

---

## Required pipeline (tough checklist)

Your `.github/workflows/` must include **all** of the following:

| # | Requirement | Tool / pattern |
|---|-------------|----------------|
| 1 | **Lint** job | Flake8 / ESLint / golangci-lint |
| 2 | **Test** job + coverage artifact or report | pytest / Jest + `actions/upload-artifact` or codecov |
| 3 | **JSON-schema contract test** | `jsonschema` / `ajv` — fails on API shape break |
| 4 | **Secret scan** | TruffleHog or gitleaks |
| 5 | **Dependency scan** | `pip-audit` / `npm audit` / `dependency-review` |
| 6 | **Trivy** image scan | `aquasecurity/trivy-action` — fail on HIGH/CRITICAL |
| 7 | **Build & push** to **public GHCR** | `docker/build-push-action` — `latest` + commit SHA tags |
| 8 | **`needs:` gating** | `build-push` only runs after lint + test pass |
| 9 | **Matrix build** | ≥2 runtime versions (e.g. Python 3.11 + 3.12) |
| 10 | **Reusable workflow** OR **composite action** | `workflow_call` or `.github/actions/...` |
| 11 | **File-size guard** | CI fails if any tracked file > 1 MB |
| 12 | **Coverage ≥ 70%** threshold in CI | fail below threshold |
| 13 | All workflows pass **`actionlint`** | no `ubntu-latest` typos |

Also required on GitHub (checked via API, **+30 pts**):

- Latest **completed** Actions run on `main` has conclusion **`success`**
- GHCR package is **public** (mentor spot-check; not auto-graded yet)

---

## How autograding works (two layers)

| Layer | What | Points |
|-------|------|--------|
| **1 — Codebase (sandbox)** | Clone your repo → `actionlint`, workflow structure, **`docker build`**, **`docker compose config`**, run tests locally in grader | **240** |
| **2 — Live GitHub** | GitHub API: Actions run for **your submitted commit**, jobs (lint/test/build) **actually passed**, GHCR package exists | **+30** |
| **3 — AI docs** | README explains pipeline stages | **+30** |

We **cannot** push to your repo on your behalf. “Live” means: after **you** push, we verify GitHub Actions really ran and passed for the **exact commit** you submitted.

### Optional fine-grained PAT (recommended)

On the submission form, paste a **read-only** fine-grained token scoped to **this repo only**:

- Actions: read
- Contents: read  
- Metadata: read
- Packages: read (for GHCR check)

Required for **private** repos. For **public** repos the server token may work, but a PAT gives reliable Actions + Packages access.

Token is encrypted, used only during grading, then removed from our database.

### Live checks (+30 breakdown)

| Check | Pts |
|-------|-----|
| Completed Actions run for submitted commit = `success` | +12 |
| lint + test + build jobs all green on that run | +10 |
| GHCR package `ghcr.io/you/repo` exists | +8 |

## Scoring (300 points)

| Part | Pts | Autograded? |
|------|-----|-------------|
| Workflows valid (`actionlint`, jobs present) | 60 | ✅ |
| Quality gates (lint, test, schema, secrets, deps, Trivy) | 90 | ✅ |
| Engineering (matrix, reusable, file-size guard, `needs:`) | 60 | ✅ |
| Docker live (`docker build`, compose ≥2 services + healthcheck, pytest) | 30 | ✅ |
| **Green Actions run on `main`** | 30 | ✅ GitHub API |
| **README / pipeline documentation** | 30 | ✅ AI rubric |
| **Total** | **300** | |

Partial credit per criterion. Resubmit anytime before the deadline — best score counts.

---

## Deployment — do you need it?

| | Required? |
|---|-----------|
| GHCR image pushed from CI | **Yes** |
| `docker compose up` works locally | **Yes** (grader checks config) |
| Deploy to VM / Fly / Cloudflare / K8s | **No** — optional portfolio stretch |
| Public URL after deploy | **No** — optional **+30 manual bonus** if you add a deploy job |

You learn deployment properly in **Week 4 (Kubernetes)**. Week 3 stops at **build + push + prove gates work**.

---

## Required repo layout

```
your-repo/
├── README.md                 # explain every pipeline stage + GHCR pull instructions
├── Dockerfile
├── compose.yaml
├── .dockerignore
├── src/  or  app/
├── tests/
│   ├── test_unit.py
│   └── test_schema.py
└── .github/
    ├── workflows/
    │   ├── ci.yml
    │   └── reusable-build.yml   # optional workflow_call
    └── actions/                 # OR composite action here
```

---

## Submission checklist

- [ ] Public GitHub repo (Week 2 app + Week 3 CI)
- [ ] All 13 pipeline requirements above
- [ ] Latest Actions run on `main` is **green**
- [ ] GHCR package **public** with `latest` + SHA tags
- [ ] `build-push` has `needs:` on quality jobs — skipped when a gate fails
- [ ] README documents each stage + links to Actions + GHCR
- [ ] Submitted on portal before deadline
- [ ] After pushing fixes, click **Update submission** to re-grade

---

## Common failures

| Mistake | Effect |
|---------|--------|
| `build-push` runs without `needs:` on gates | −9 to −15 |
| No Trivy / secret scan / schema test | −6 each |
| Private GHCR package | Actions +30 may pass but mentor may dock |
| Only one compose service | −9 |
| No healthcheck in compose | −6 |
| Workflows fail `actionlint` | −9 to −15 |
| No green Actions run on `main` | **−30** (bonus not awarded) |
| Thin README | **−30** (AI docs score) |

---

## Program arc

```text
Week 2  →  Dockerized app (Dockerfile, compose)       ✓ prerequisite
Week 3  →  THIS SUBMISSION (CI pipeline + GHCR)       ← you are here
Week 4  →  Deploy GHCR image to Kubernetes
```

*Questions: `#devops-help` · Spec version: 2026 cohort*
