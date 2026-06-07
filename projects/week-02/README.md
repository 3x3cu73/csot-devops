# Week 02 — Mini-Project submission guide

**Project**: Dockerized App Stack (multi-service `docker compose` app)
**Track**: 🟢 Local Track · **50 points** · manually graded
**Deadline**: **Sunday 11:59 PM (IST)** of Week 2
**Spec**: [content/week-02-docker-compose-debugging.md → Weekly Mini-Project](../../content/week-02-docker-compose-debugging.md#weekly-mini-project--dockerized-app-stack)

> Two different things land in Week 2, **don't confuse them**:
>
> | | Contest (autograded) | Mini-Project (this guide) |
> |---|---|---|
> | **What** | 12 Docker/Compose incident tasks | Dockerized App Stack repo |
> | **Points** | 200 | 50 |
> | **How submitted** | `csot` CLI / `csot challenge` → [csot-devops.devclub.in](https://csot-devops.devclub.in) | **Submission form** (link shared in your cohort group) |
> | **Grading** | Automatic, in a Docker sandbox | Manual review by mentors |
> | **Guide** | [`/submission/README.md`](../../submission/README.md) | You're reading it |

---

## 1. What you're submitting

A **public GitHub repository** containing a multi-service app you can bring up with a single `docker compose up --build` (everything specified under **[Weekly Mini-Project — Dockerized App Stack](../../content/week-02-docker-compose-debugging.md#weekly-mini-project--dockerized-app-stack)** in the Week 2 content guide). At minimum:

- `Dockerfile` — multi-stage or slim base, runs as **non-root**
- `.dockerignore` — no secrets/caches baked into the image
- `compose.yaml` — app + database (+ optional Redis), wired by service name
- `README.md` — exact `docker compose up` instructions + screenshots

Full required behavior, the file list, and the **grading rubric (50 pts)** live in the [Weekly Mini-Project section of the Week 2 guide](../../content/week-02-docker-compose-debugging.md#weekly-mini-project--dockerized-app-stack). Read it before you start — this file is only about *how to hand it in*.

---

## 2. Submission checklist (do these before opening the form)

- [ ] Project repo is **public** on GitHub
- [ ] `docker compose up --build` starts the whole stack cleanly from scratch
- [ ] App reachable on a documented `localhost` port
- [ ] Services talk to each other **by service name** (not `localhost` / hardcoded IPs)
- [ ] Database data persists in a **named volume** across `docker compose down` + `up`
- [ ] All config comes from **environment variables** (no secrets in the image)
- [ ] A working **healthcheck** on the app or database
- [ ] App process runs as **non-root**; logs go to **stdout/stderr**
- [ ] `.dockerignore` committed; `.env` **not** committed (check `.gitignore`)
- [ ] `trufflehog git file://. --only-verified` → **clean** (no verified secrets in history)
- [ ] `README.md` includes a 1-line "How to run" and a screenshot of the running stack
- [ ] You know the **exact commit SHA** you want graded (`git rev-parse HEAD` in the project repo)

---

## 3. How to submit (form flow)

We submit Week 2 mini-projects via a **submission form**. The form link is shared in your cohort group on Discord / WhatsApp — it is **not posted in this public repo**.

> **Prerequisite:** you've already forked this repo per the root README's **[How to Participate](../../README.md#-how-to-participate-read-this-once-then-refer-back)** section. Your Dockerized App Stack lives in **its own separate public GitHub repo** — *not* inside this fork.

### Step-by-step

```bash
# 1. Build your stack in its own dedicated public repo, e.g.:
#    github.com/<your-github-username>/dockerized-app-week02

# 2. When you're ready to submit, grab the commit SHA you want graded:
cd ~/path/to/dockerized-app-week02
git rev-parse HEAD
# → 9f3c1a8b...   ← copy this

# 3. Make sure the repo is PUBLIC and the README renders correctly on GitHub.

# 4. Open the submission form (link in your cohort Discord / WhatsApp group)
#    and fill in the fields listed below.
```

### What the form will ask for

Have these ready **before** you open the form, so you fill it out in one go:

| Field | Example | Notes |
|---|---|---|
| Name | `Jane Doe` | As registered with DevClub |
| DevClub email | `jane@example.com` | The email tied to your DevClub login |
| GitHub username | `janedoe` | Without the `@` |
| Project repo URL (public) | `https://github.com/janedoe/dockerized-app-week02` | Must be **public** at grading time |
| Commit SHA being submitted | `9f3c1a8` | Short or full SHA, from `git rev-parse HEAD` |
| Project chosen | `Default (Dockerized App Stack)` or alternative name | See [Alternative Mini-Project Ideas](../../content/week-02-docker-compose-debugging.md#alternative-mini-project-ideas) |
| What you built (2–4 sentences) | "Flask API + Postgres + Redis via compose, multi-stage non-root image, named volume for the DB, healthchecks on app and db." | Plain text, brief |
| Anything for the reviewer | Edge cases, known issues, what you'd do with more time | Optional but encouraged |

After you submit, a mentor will review within ~3 days and post your score in the cohort group / on the leaderboard.

### Important rules

- You may **re-submit** the form before the deadline — the **latest submission** before deadline is the one graded. Push fixes first, then re-submit with the new SHA.
- **Do not change the repo's visibility to private** between submission and grading — that breaks review and forfeits your points.
- **Do not force-push or rewrite history** after submitting — the grader checks out the exact SHA you submitted.

---

## 4. Late, partial, or "I'm stuck" submissions

- **Late**: Submit anyway — capped at 50% credit for the mini-project. Better than zero.
- **Partial**: Submit what works. Be honest in the *"Anything for the reviewer"* field of the form. Partial credit is real.
- **Stuck**: Ask in your cohort group `#help` channel describing what you tried. Mentors check daily.

---

## 5. Where things live (quick map)

```text
csot-devops/                                       ← upstream (this repo) — read-only for you
├── content/week-02-docker-compose-debugging.md    ← teaching material + project spec + rubric
├── submission/
│   ├── README.md                                  ← CONTEST submission guide (csot CLI)
│   └── week-02/                                   ← your contest task files (in YOUR fork)
└── projects/
    ├── README.md                                  ← index of all weekly project guides
    └── week-02/
        └── README.md                              ← (you are here) mini-project submission guide

github.com/<you>/dockerized-app-week02/            ← YOUR project repo (separate, public)
                                                   ← URL goes into the submission form
```

---

## 6. FAQ

**Q. Where is the submission form link?**
In your cohort group (Discord / WhatsApp), pinned by an organizer. It's intentionally not posted here so only registered participants can submit.

**Q. Can my project just live inside my fork of this repo (in `projects/week-02/`)?**
Technically yes, but it's a bad portfolio piece. Create a **dedicated public repo** for the stack and submit *that* URL. Your fork is the workspace; the dedicated repo is the deliverable.

**Q. Can I submit the contest tasks via this form?**
No. Contest tasks go through the `csot` CLI / `csot challenge` — see [`/submission/README.md`](../../submission/README.md). They are graded automatically and separately.

**Q. Can I pick an Alternative Mini-Project instead?**
Yes — see [Alternative Mini-Project Ideas](../../content/week-02-docker-compose-debugging.md#alternative-mini-project-ideas) in the Week 2 guide. Note which one you picked in the form's *"Project chosen"* field; the same 50-point rubric applies (criteria mapped sensibly to your chosen project).

**Q. I submitted, then noticed a bug. Can I fix it?**
Yes — push the fix to your project repo, then re-submit the form with the new commit SHA. The latest submission before the deadline is the one graded.
