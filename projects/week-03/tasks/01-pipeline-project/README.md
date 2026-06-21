# Week 3 Project — Production CI/CD Pipeline

## How to submit

1. Push your Week 2 + Week 3 work to a **public** GitHub repo.
2. Confirm the latest Actions run on **`main`** is green.
3. Go to **[csot-devops.devclub.in/submission](https://csot-devops.devclub.in/submission)** → **Week 3**.
4. Paste your repo URL (folder `.` unless the project is in a subfolder).
5. Wait for autograding (usually 2–10 minutes). **Resubmit** after pushing fixes.

There is **no** `csot submit` for Week 3.

---

## What the autograder checks (240 pts base)

Runs against a **clone of your repo** at the submitted commit:

### Workflows (static)

- All `.github/workflows/*.{yml,yaml}` pass **`actionlint`**
- No runner typos (`ubuntu-latest`, not `ubntu-latest`)
- Jobs: **lint**, **test** (with coverage artifact), **secret scan**, **dependency scan**, **schema/contract test**, **Trivy**
- **`docker/build-push-action`** present
- **Build job gated** with `needs:` on lint/test
- **Matrix** with ≥2 runtime versions
- **Reusable workflow** (`workflow_call`) or **composite action**
- **File-size guard** step (>1 MB fail)

### Live Docker + tests

- `docker build` succeeds using your **Dockerfile**
- `docker compose config` valid with **≥2 services** and a **healthcheck**
- **pytest** passes locally with **≥70% coverage**

### GitHub + AI (+60 pts)

| Check | Pts |
|-------|-----|
| Completed Actions run for submitted commit = **success** | +12 |
| lint + test + build jobs green on GitHub | +10 |
| GHCR package exists | +8 |
| README / pipeline documentation (AI rubric) | +30 |

---

## Symptoms if you're losing points

| Symptom in grader log | Likely fix |
|-----------------------|------------|
| `actionlint` errors | Fix YAML syntax, action versions, expression typos |
| `build-push must needs:` | Add `needs: [lint, test, ...]` on build job |
| `compose needs app + database` | Add DB service to compose |
| `No completed Actions runs` | Push to `main`, wait for workflow, ensure it passes |
| Low AI docs score | README: explain each stage, link Actions + GHCR, how to run locally |

---

## Deployment

**Not required.** GHCR push in CI is enough. Deploy comes in Week 4 (Kubernetes).

---

## Points: **300**

See [`week-03-projects.md`](../../../../week-03-projects.md) in the curriculum repo for the full student rubric.
