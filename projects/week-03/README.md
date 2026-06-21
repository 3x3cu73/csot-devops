# Week 3 — Pipeline Project (single submission)

One project, one autograder run. Submit your **public GitHub repo** on the [submission portal](https://csot-devops.devclub.in/submission) — the server clones it, runs live checks, and scores up to **300 points**.

**No `csot submit` tarball.** No 12 separate tasks.

Open the project spec: **[01-pipeline-project](./tasks/01-pipeline-project/README.md)**

## What gets graded automatically

| Layer | Points | How |
|-------|--------|-----|
| Repo + workflows + Docker | 240 | Grader: `actionlint`, workflow structure, `docker build`, `compose config`, pytest |
| Green GitHub Actions on `main` | +30 | Server checks run for submitted commit via GitHub API |
| README / pipeline docs (AI rubric) | +30 | Server AI review of documentation quality |
| **Total** | **300** | |

## Do students need to deploy?

**No.** Deployment is **not required** for full marks.

Required: public repo, working **Dockerfile + compose**, **GitHub Actions** pipeline, **GHCR push** in CI, green run on `main`.

**Optional +30 bonus** (manual / next iteration): deploy stage with a public URL after GHCR push — not part of the 300-point autograder.

## Resubmit

Push fixes to GitHub, then hit **Update submission** on the portal. Each resubmit re-queues autograding on the latest commit.
