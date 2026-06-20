# Week 3 — What To Do (Student Guide)

> **Theme**: Automate everything from commit to artifact. Never ship broken or insecure builds.
>
> **Time budget**: ~8 hours (reading 2.5h + builds 3h + project polish 2.5h)
>
> **Deep reference** (modules, YAML, expected output): [`content/week-02-cicd-quality-registries.md`](./content/week-02-cicd-quality-registries.md)  
> **Submissions & grading**: [`week-03-projects.md`](./week-03-projects.md)

---

## Table of Contents

1. [Where Week 3 sits in the program](#where-week-3-sits-in-the-program)
2. [What you need before starting](#what-you-need-before-starting)
3. [Learning outcomes](#learning-outcomes)
4. [Recommended pace (Mon → Sun)](#recommended-pace-mon--sun)
5. [Module map — what to read and when](#module-map--what-to-read-and-when)
6. [Build 1 — PR pipeline with 4 quality gates (Wed)](#build-1--pr-pipeline-with-4-quality-gates-wed)
7. [Build 2 — Build & push to GHCR (Fri)](#build-2--build--push-to-ghcr-fri)
8. [Optional: Green Build Race (Sat)](#optional-green-build-race-sat)
9. [Optional stretch — deploy job](#optional-stretch--deploy-job)
10. [Sunday: polish for submission](#sunday-polish-for-submission)
11. [Quiz prep topics](#quiz-prep-topics)

---

## Where Week 3 sits in the program

Week 3 is **CI/CD week**. Docker was Week 2 — you do not re-learn containers here.

```text
Week 1  →  Linux, shell, Git, nginx, secrets hygiene
Week 2  →  Docker, Compose, container debugging     ✓ prerequisite
Week 3  →  GitHub Actions, quality gates, GHCR      ← this week
Week 4  →  Kubernetes, Helm, GitOps
```

| From | You reuse this week |
|------|---------------------|
| **Week 1** | Git hygiene, `.env.example`, TruffleHog-clean history, conventional commits |
| **Week 2** | Your containerized app repo (`Dockerfile`, `compose.yaml`, working image) |
| **Week 3 (new)** | GitHub Actions pipeline, quality gates, GHCR push, Trivy/Syft in CI |

**Ideal path:** take your **Week 2 Docker repo** and add a production-grade `.github/workflows/` pipeline on top. Submit that same repo for Week 3.

---

## What you need before starting

- [ ] Week 1 done — comfortable with Git, branching, PRs
- [ ] Week 2 done — public repo with `Dockerfile` + `compose.yaml` that runs locally
- [ ] GitHub account; repo is **public** (or will be for submission)
- [ ] No new local installs required — CI runs in GitHub's cloud
- [ ] (Optional) `act` installed if you want to test workflows locally

---

## Learning outcomes

By the end of this week you can:

- Explain CI vs CD and why pipelines exist
- Write GitHub Actions workflows (triggers, jobs, steps, `needs:`, secrets)
- Enforce **lint → test → schema → secret scan** before merge
- Build and push Docker images to **GHCR** from CI
- Add **Trivy** and **Syft SBOM** steps to the pipeline
- Use matrix builds, caching, and reusable workflows
- Configure branch protection so `main` stays green

---

## Recommended pace (Mon → Sun)

| When | Read | Do |
|------|------|-----|
| **Mon** | Modules 1–2 ([CI/CD doc](./content/week-02-cicd-quality-registries.md)) | Skim your Week 2 repo; ensure `Dockerfile` builds |
| **Tue** | Modules 3–4 | Draft `.github/workflows/ci.yml` with lint + test |
| **Wed** | Build 1 | **Build 1** — 4 quality gates + branch protection |
| **Thu** | Modules 5–6 | Add GHCR login + `docker/build-push-action` |
| **Fri** | Build 2 | **Build 2** — push to GHCR only after gates pass |
| **Sat** | Modules 7–8 + optional contest | Trivy, Syft, matrix, releases; optional Green Build Race |
| **Sun** | — | Finish full pipeline per [projects doc](./week-03-projects.md); **submit on portal** |

---

## Module map — what to read and when

Full teaching content: [`content/week-02-cicd-quality-registries.md`](./content/week-02-cicd-quality-registries.md)

| Module | Topic | Do this after reading |
|--------|-------|------------------------|
| **1** | Why CI/CD | Draw your pipeline stages on paper |
| **2** | GitHub Actions anatomy | Push a minimal `ci.yml` that echoes hello |
| **3** | Workflow patterns | Add caching + one matrix job |
| **4** | Quality gates | lint, test, JSON-schema, TruffleHog |
| **5** | GHCR delivery | `docker/build-push-action` on `main` |
| **6** | Releases & hygiene | semantic-release or release-please (optional until Sun) |
| **7** | Jenkins awareness | Read only — interview prep |
| **8** | Other CI tools | Skim GitLab CI / CircleCI comparison |

**Supply chain (used in CI this week):** add Trivy + Syft steps — see [Module 5 in week-03-containers-compose-security.md](./content/week-03-containers-compose-security.md#module-5--image-security--supply-chain) for scan commands only.

---

## Build 1 — PR pipeline with 4 quality gates (Wed)

**Goal:** GitHub Actions on every PR: **lint → test → JSON-schema → TruffleHog**. Branch protection blocks merge on red.

**Summary steps:**

1. Start from your Week 2 repo (or the starter layout in the [CI/CD Build 1 section](./content/week-02-cicd-quality-registries.md#build-1--pr-pipeline-with-4-quality-gates-wed)).
2. Add `.github/workflows/ci.yml` with four jobs.
3. Enable branch protection on `main` — require all four checks.
4. Prove each gate works by breaking it once (four screenshots).

**Done when:** all four jobs run on PRs; each can fail independently; branch protection enforced.

---

## Build 2 — Build & push to GHCR (Fri)

**Goal:** On push to `main`, build your Week 2 Docker image and push to **public GHCR** — but **only if all quality gates pass**.

**Summary steps:**

1. Ensure `Dockerfile` exists (from Week 2).
2. Add reusable workflow or inline `docker/build-push-action` job with `needs: [lint, test, schema-check, secret-scan]`.
3. Tags: `latest` + `sha-<short>`.
4. Verify: `docker pull ghcr.io/<you>/<repo>:latest` and run it.
5. Screenshot a failed gate showing **skipped** `build-push`.

Full YAML: [CI/CD Build 2 section](./content/week-02-cicd-quality-registries.md#build-2--build--push-to-ghcr-fri).

**Done when:** GHCR image public and pullable; bad code does not produce a new image.

---

## Optional: Green Build Race (Sat)

Fix a broken workflow, add security/build stages, optimize runtime. Details in the [CI/CD contest section](./content/week-02-cicd-quality-registries.md#weekly-contest--green-build-race). **Optional** — not part of the 100-point Week 3 submission.

---

## Optional stretch — deploy job

After GHCR push, add a deploy job (SSH to a VM, Fly.io, or Cloudflare Pages). Counts toward the **+10 cloud bonus** in [`week-03-projects.md`](./week-03-projects.md#optional-cloud-bonus-10-pts). Full YAML examples in the [CI/CD Part B section](./content/week-02-cicd-quality-registries.md#-part-b--cloud-track-optional-bonus-10-pts).

**Week 4 preview (not required):** do not add Kubernetes this week — that is next week's full topic.

---

## Sunday: polish for submission

Your **mandatory deliverable** is a **public GitHub repo** with a complete CI pipeline (not `csot submit` files).

1. Complete all [9 pipeline features](./week-03-projects.md#default-project--production-grade-ci-pipeline) in the projects doc.
2. Confirm latest Actions run is green on `main`.
3. Confirm GHCR package is **public**.
4. Submit on [csot-devops.devclub.in/submission](https://csot-devops.devclub.in/submission) → Week 3.

---

## Quiz prep topics

- Pipeline stages and their order
- GitHub Actions: workflows, jobs, steps, triggers
- Matrix builds and caching
- Secrets vs `GITHUB_TOKEN`
- Reusable workflows vs composite actions
- GHCR vs Docker Hub
- JSON-schema contract testing — why it matters
- Conventional commits + semantic-release
- Branch protection rules
- Trivy in CI — what HIGH/CRITICAL means

---

**Next:** open [`week-03-projects.md`](./week-03-projects.md) for the 100-point rubric and submission checklist.
