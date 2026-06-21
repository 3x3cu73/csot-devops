# Week 3 — Projects & Submissions

> **Theme**: Production-grade CI/CD pipeline for your Week 2 containerized app.
>
> **Submission type:** public **GitHub repository** (+ portal form) — not `csot submit` file uploads.
>
> **Deadline:** Sunday 11:59 PM (IST)
>
> **Points:** 100 (mandatory) + optional 10 (cloud bonus)
>
> **What to learn / build this week:** [`week-03-guide.md`](./week-03-guide.md)  
> **Module deep-dive:** [`content/week-02-cicd-quality-registries.md`](./content/week-02-cicd-quality-registries.md)

---

## Table of Contents

1. [Submission type](#submission-type)
2. [Prerequisites (Week 2)](#prerequisites-week-2)
3. [Default project — Production-Grade CI Pipeline](#default-project--production-grade-ci-pipeline)
4. [Scoring overview (100 pts)](#scoring-overview-100-pts)
5. [Part A — Quality gates (25 pts)](#part-a--quality-gates-25-pts)
6. [Part B — Security in CI (15 pts)](#part-b--security-in-ci-15-pts)
7. [Part C — GHCR build & push (25 pts)](#part-c--ghcr-build--push-25-pts)
8. [Part D — Pipeline engineering (20 pts)](#part-d--pipeline-engineering-20-pts)
9. [Part E — Documentation & proof (15 pts)](#part-e--documentation--proof-15-pts)
10. [Required repo layout](#required-repo-layout)
11. [How to submit](#how-to-submit)
12. [Submission checklist](#submission-checklist)
13. [Alternative project ideas](#alternative-project-ideas)
14. [Optional cloud bonus (+10 pts)](#optional-cloud-bonus-10-pts)
15. [Common failures](#common-failures)
16. [Self-check before submit](#self-check-before-submit)

---

## Submission type

| | |
|---|---|
| **Deliverable** | One **public GitHub repo** with a green CI pipeline + GHCR image |
| **Portal** | [csot-devops.devclub.in/submission](https://csot-devops.devclub.in/submission) → Week 3 |
| **What you paste** | Repo URL, link to latest **green** Actions run, GHCR image URL |
| **Not accepted** | Private repos, repos with no `.github/workflows/`, pipelines that push on red |

Mentor smoke test (~10 minutes):

1. Open your latest green Actions run on `main`.
2. Confirm required jobs passed (lint, test, schema, secret scan, build-push).
3. `docker pull ghcr.io/<you>/<repo>:latest` and run the container.
4. (Sanity) `docker compose up -d` still works if you kept Week 2 compose files.

---

## Prerequisites (Week 2)

Your repo should **already** have from Week 2 (not re-graded in depth, but must exist):

| Item | Required |
|------|----------|
| `Dockerfile` | Builds your app image |
| `compose.yaml` | Multi-service stack (app + DB + cache, etc.) |
| App runs locally | `docker compose up --build` succeeds |

Week 3 adds **CI/CD on top** — you are not submitting Docker learning again.

---

## Default project — Production-Grade CI Pipeline

Take your **Week 2 containerized app** and add a complete GitHub Actions pipeline.

### Required pipeline features (all 9)

| # | Feature | Tool |
|---|---------|------|
| 1 | **Lint** | ESLint / Flake8 / golangci-lint |
| 2 | **Test** + coverage artifact | pytest / Jest / go test |
| 3 | **JSON-schema contract test** | `jsonschema` / `ajv` |
| 4 | **Secret scan** | TruffleHog or gitleaks |
| 5 | **Dependency scan** | `pip-audit` / `npm audit` / `govulncheck` |
| 6 | **Build & push** to GHCR (`latest` + commit SHA) | `docker/build-push-action` |
| 7 | **Auto-tag releases** on merge to `main` | `semantic-release` or `release-please` |
| 8 | **Matrix build** — ≥ 2 language/runtime versions | matrix strategy |
| 9 | **File-size guard** — fail if any file > 1 MB | custom step |

Also required:

- **Trivy** scan in CI — fail on HIGH/CRITICAL (image built in pipeline)
- **Syft SBOM** uploaded as CI artifact or Release asset
- **Reusable workflow** (`workflow_call`) or **composite action**
- **Branch protection** on `main` requiring status checks
- **Two failure screenshots** — one schema break, one secret-scan break (proves gates work)

---

## Scoring overview (100 pts)

| Part | Focus | Points |
|------|-------|--------|
| **A** | Quality gates (lint, test, schema, secrets) | 25 |
| **B** | Security in CI (dependency scan, Trivy, SBOM) | 15 |
| **C** | GHCR build & push | 25 |
| **D** | Pipeline engineering | 20 |
| **E** | Documentation & proof | 15 |
| | **Total** | **100** |

**Auto-fail:** verified secret in Git history; image pushed when a quality gate failed; GHCR package private.

Partial credit per criterion below.

---

## Part A — Quality gates (25 pts)

| Criterion | Pts | Pass condition |
|-----------|-----|----------------|
| Lint job | 6 | Runs on PR + push; fails on style violations |
| Test job | 7 | Runs tests; uploads coverage artifact |
| JSON-schema contract test | 6 | Fails when API response shape changes |
| Secret scan | 6 | TruffleHog/gitleaks on PR; fails on verified secret |

---

## Part B — Security in CI (15 pts)

| Criterion | Pts | Pass condition |
|-----------|-----|----------------|
| Dependency scan | 5 | `pip-audit` / `npm audit` / govulncheck in workflow |
| Trivy image scan | 5 | Fails CI on HIGH/CRITICAL (`ignore-unfixed` OK) |
| SBOM artifact | 5 | Syft output in Actions artifacts or Releases |

---

## Part C — GHCR build & push (25 pts)

| Criterion | Pts | Pass condition |
|-----------|-----|----------------|
| Builds Docker image in CI | 8 | Uses Week 2 `Dockerfile` via build-push action |
| Pushes to public GHCR | 8 | `ghcr.io/<you>/<repo>:latest` + `:sha-*` exist |
| Gated by quality jobs | 6 | `build-push` has `needs:` on lint/test/schema/secret — skipped on red |
| Image pullable | 3 | README or mentor can `docker pull` and run |

---

## Part D — Pipeline engineering (20 pts)

| Criterion | Pts | Pass condition |
|-----------|-----|----------------|
| Matrix build | 5 | ≥ 2 versions (e.g. Python 3.11 + 3.12) |
| Reusable workflow or composite action | 5 | `workflow_call` or `.github/actions/...` |
| File-size guard | 4 | CI fails if any tracked file > 1 MB |
| Auto releases | 6 | semantic-release or release-please creates/tags on `main` |

---

## Part E — Documentation & proof (15 pts)

| Criterion | Pts | Pass condition |
|-----------|-----|----------------|
| README — pipeline docs | 5 | Explains each stage; links to Actions + GHCR |
| Branch protection | 3 | Screenshot or settings showing required checks |
| Green pipeline screenshot | 3 | Latest `main` run all green |
| Failure proof | 4 | Screenshots: schema gate failed + secret gate failed |

---

## Required repo layout

```
your-week3-repo/                    ← same repo as Week 2 Docker project
├── README.md                       # pipeline docs + GHCR pull instructions
├── Dockerfile                      # from Week 2
├── compose.yaml                    # from Week 2
├── .dockerignore
├── src/                            # or app/
├── tests/
│   ├── test_unit.py
│   ├── test_schema.py
│   └── *.schema.json
├── requirements.txt                # or package.json
└── .github/
    ├── workflows/
    │   ├── ci.yml                  # main pipeline
    │   └── reusable-build-push.yml # optional reusable workflow
    └── actions/                    # optional composite action
```

---

## How to submit

1. Push final code to `main` on a **public** repo.
2. Confirm latest Actions run on `main` is **green**.
3. Confirm GHCR package visibility is **public** (Packages → Package settings).
4. Go to **[csot-devops.devclub.in/submission](https://csot-devops.devclub.in/submission)** → **Week 3**.
5. Submit:
   - GitHub repo URL
   - Link to latest green Actions run on `main`
   - GHCR image URL (`ghcr.io/...`)

Grading: 3–5 business days. Disputes → `#devops-help` with repo link + part ID.

---

## Submission checklist

- [ ] Public GitHub repo (Week 2 Docker app + Week 3 CI)
- [ ] All 9 pipeline features working
- [ ] Trivy + Syft in CI
- [ ] Reusable workflow or composite action
- [ ] Branch protection on `main`
- [ ] GHCR public with `latest` + SHA tags
- [ ] `build-push` skipped when a gate fails (screenshot)
- [ ] Schema + secret scan failure screenshots
- [ ] Green `main` pipeline screenshot
- [ ] `trufflehog git file://. --only-verified` clean
- [ ] Portal submitted before deadline

---

## Alternative project ideas

Same **100-point rubric** — pipeline must still lint, test, scan, and push a container image to GHCR. Get mentor approval if scope differs.

| # | Idea |
|---|------|
| 1 | **PR bot** — Action comments lint/coverage diff on every PR |
| 2 | **Release-notes generator** — conventional commits → CHANGELOG + GitHub Release |
| 3 | **CI template pack** — reusable workflows for Node/Python/Go |
| 4 | **Integration test job** — CI runs `docker compose up` + hits API endpoints |
| 5 | **LLM eval pipeline** — schema-validate API responses from your app |

Full list: [CI/CD doc § alternatives](./content/week-02-cicd-quality-registries.md#alternative-mini-project-ideas).

---

## Optional cloud bonus (+10 pts)

Outside the 100-point base. Add a **deploy** stage after GHCR push.

| Criterion | Pts |
|-----------|-----|
| Deploy job updates a running app (SSH, Fly.io, or Cloudflare Pages) | 4 |
| Image pinned by commit SHA (not `latest`) | 2 |
| GitHub Environment with manual approval for production | 2 |
| Public URL works after deploy | 2 |

Examples: [CI/CD Part B — Cloud Track](./content/week-02-cicd-quality-registries.md#-part-b--cloud-track-optional-bonus-10-pts).

---

## Common failures

| Mistake | Cost |
|---------|------|
| `build-push` runs without `needs:` on gates | Part C −6 |
| GHCR package left private | Part C −8 |
| No failure screenshots | Part E −4 |
| No branch protection | Part E −3 |
| Matrix job only on one version | Part D −5 |
| Pushing on red (demo secret still in branch) | **Auto-fail** |
| Secret in Git history | **0/100** |

---

## Self-check before submit

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Week 3 CI pre-submit ==="
test -f Dockerfile && echo "✓ Dockerfile (Week 2)"
test -f compose.yaml && echo "✓ compose.yaml (Week 2)"
test -d .github/workflows && echo "✓ workflows dir"
ls .github/workflows/

echo "Manual checks:"
echo "  1. Open Actions → latest main run → all green"
echo "  2. build-push has needs: [lint, test, ...]"
echo "  3. ghcr.io/<you>/<repo> public + pullable"
echo "  4. Branch protection requires status checks"
echo "  5. Failure screenshots in README or DEMO.md"
trufflehog git file://. --only-verified 2>/dev/null | tail -3 || true
```

---

## Program arc

```text
Week 1  →  DevOps Toolkit (scripts, systemd, nginx)
Week 2  →  Dockerized app (Dockerfile, compose)       ✓ prerequisite
Week 3  →  THIS SUBMISSION (CI pipeline + GHCR)     ← you are here
Week 4  →  Deploy GHCR image to Kubernetes
```

---

*Spec version: 2026 cohort · Questions: `#devops-help`*
