# Week 3 — What To Do (Student Guide)

> **One project, one submission, autograded to 300 points.**
>
> **Deep reference:** [`content/week-02-cicd-quality-registries.md`](./content/week-02-cicd-quality-registries.md)  
> **Project spec & rubric:** [`week-03-projects.md`](./week-03-projects.md)  
> **Contest portal:** [csot-devops.devclub.in/submission](https://csot-devops.devclub.in/submission)

---

## The one deliverable

Take your **Week 2 Docker repo** and add a production-grade GitHub Actions pipeline. Submit the **public repo URL** on the portal — not `csot submit` files.

The server will:

1. Clone your repo at the latest commit on `main`
2. Run **live checks** (`actionlint`, `docker build`, `compose config`, pytest)
3. Verify **GitHub Actions** is green on `main` (+30)
4. Run an **AI review** of your README (+30)

**Max score: 300 points.**

---

## Do I need deployment?

**No.** Week 3 ends at **CI + GHCR**. You do **not** need a public deploy URL for full marks.

Optional stretch: add a deploy job (SSH / Fly.io / Cloudflare) for portfolio polish — not autograded.

---

## Recommended pace

| When | Do |
|------|-----|
| **Mon–Tue** | Read CI/CD modules; draft `.github/workflows/ci.yml` with lint + test |
| **Wed** | Add 4 quality gates + branch protection |
| **Thu–Fri** | GHCR push, Trivy, Syft, matrix, reusable workflow |
| **Sat** | Fix until Actions is green on `main`; polish README |
| **Sun** | Submit on portal; verify autograder score; resubmit if needed |

---

## Before you submit

```bash
# Local sanity
actionlint .github/workflows/*.yml
docker compose config
docker compose up --build -d
docker build -t test .

# Secrets
trufflehog git file://. --only-verified
```

Then paste repo URL on the portal → Week 3 → **Submit**.

---

**Next:** [`week-03-projects.md`](./week-03-projects.md) for the full tough rubric.
