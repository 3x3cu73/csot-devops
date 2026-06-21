# Week 3 — CI/CD, Quality Engineering & Registries

> **Curriculum note:** This is **Week 3** material. The filename `week-02-cicd-...` is legacy from an earlier ordering — **Week 2 is Docker** ([`week-02-docker-compose-debugging.md`](./week-02-docker-compose-debugging.md)).

> **Theme**: Automate everything from commit to artifact. Never let a broken or insecure build ship.
>
> **Time Budget**: ~8 hours (reading 2.5h + builds 3h + mini-project 2.5h)

---

## Table of Contents

1. [Tools & Software Used](#tools--software-used)
2. [Prerequisites](#prerequisites)
3. [Learning Outcomes](#learning-outcomes)
4. [Module 1 — Why CI/CD?](#module-1--why-cicd)
5. [Module 2 — GitHub Actions Anatomy](#module-2--github-actions-anatomy)
6. [Module 3 — Workflow Patterns](#module-3--workflow-patterns)
7. [Module 4 — Quality Gates](#module-4--quality-gates)
8. [Module 5 — Continuous Delivery: GHCR](#module-5--continuous-delivery-ghcr)
9. [Module 6 — Repo Hygiene & Release Automation](#module-6--repo-hygiene--release-automation)
10. [Module 7 — Jenkins Awareness](#module-7--jenkins-awareness)
11. [Module 8 — Other CI Tools (Brief)](#module-8--other-ci-tools-brief)
12. [Build 1 — PR Pipeline with 4 Quality Gates (Wed)](#build-1--pr-pipeline-with-4-quality-gates-wed)
13. [Build 2 — Build & Push to GHCR (Fri)](#build-2--build--push-to-ghcr-fri)
14. [Weekly Mini-Project — Production-Grade CI Pipeline](#weekly-mini-project--production-grade-ci-pipeline)
15. [Alternative Mini-Project Ideas](#alternative-mini-project-ideas)
16. [Weekly Quiz Topics](#weekly-quiz-topics)
17. [Weekly Contest — Green Build Race](#weekly-contest--green-build-race)
18. [Resources](#resources)

---

## Tools & Software Used

This week lives almost entirely in GitHub's cloud. **No new local installs required.**

| Tool | Purpose | Setup |
|------|---------|-------|
| **GitHub Actions** | CI/CD orchestration | Built into GitHub (free 2000 min/month) |
| **GitHub Container Registry (GHCR)** | Container image storage | Built into GitHub, free for public images |
| **TruffleHog** | Secret scanning | Run as an Action (no install) |
| **gitleaks** | Alternative secret scanner | Run as an Action |
| **Trivy** | Vulnerability scanner | Run as an Action in your pipeline this week |
| **semantic-release** | Auto-tag releases from conventional commits | Run as an Action |
| **release-please** | Google's CHANGELOG/release automation | Run as an Action |
| **Dependabot** | Dependency updates | Enable via `.github/dependabot.yml` |
| **ESLint / Flake8 / golangci-lint** | Linting | Project-dependent |
| **pytest / Jest / go test** | Testing | Project-dependent |
| **`jsonschema` (Python) / `ajv` (JS)** | API contract testing | `pip install jsonschema` / `npm install ajv` |
| **act** (optional) | Run GitHub Actions locally | `brew install act` or [github.com/nektos/act](https://github.com/nektos/act) |

**Optional local tool — `act`** — lets you test Actions on your laptop without push-test-push cycles:

```bash
# Linux:
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Run a workflow locally:
act -j build
```

---

## Prerequisites

- Week 1 completed (Git fluency, a working web app)
- The Build 2 app from Week 1 (Python/Node web app under systemd + nginx)
- GitHub account with at least one repo

---

## Learning Outcomes

By end of week, you can:
- Explain why CI/CD exists and the cost of not having it
- Write production-grade GitHub Actions workflows from scratch
- Use matrix builds, caching, secrets, environments, reusable workflows
- Enforce quality with linting, testing, **JSON schema contract testing**, and **secret scanning**
- Push container images to **GHCR** from CI
- Recognize Jenkins workflows and know when companies use them
- Auto-tag releases using conventional commits

---

## Module 1 — Why CI/CD?

### The Pre-CI/CD Pain Story

```
Monday:  Alice merges her feature.
Tuesday: Bob merges his feature.
Wednesday: Release day. Someone runs the build manually on their laptop.
           "Wait, the tests don't pass." "Which tests?" "I don't know."
           "Works on my machine." 
           3 hours of debugging. Ship at midnight. Outage at 2 AM.
```

CI/CD turns this into:

```
Monday:  Alice opens a PR. Pipeline runs in 5 min. Tests pass. Merged.
         Within 1 minute, the change is live in staging.
Tuesday: Bob opens a PR. Pipeline catches a regression. Bob fixes it. Merged.
         Live in staging in 1 min.
Wednesday: One-click promotion to prod. Done.
```

### The Vocabulary

| Term | Meaning |
|---|---|
| **CI** (Continuous Integration) | Every commit is built and tested automatically |
| **CD** (Continuous Delivery) | Every passing build is deployable on demand |
| **CD** (Continuous Deployment) | Every passing build is auto-deployed |
| **Pipeline** | The sequence: lint → test → build → scan → deploy |
| **Stage** | One logical phase of the pipeline |
| **Job** | A unit of work in a stage |
| **Artifact** | The output (binary, image, zip) of a build |
| **Runner** | The machine that executes the pipeline |
| **Trigger** | What kicks off the pipeline (push, PR, schedule) |

### Pipeline Stages (the canonical order)

```
[code commit]
     ↓
   LINT           ← code style check (fast, fail loud)
     ↓
   TEST           ← unit + integration tests
     ↓
  SCAN            ← security (TruffleHog, dependency audit)
     ↓
   BUILD          ← compile / package / docker build
     ↓
   PUSH           ← upload artifact to registry
     ↓
  DEPLOY          ← apply to staging / prod
```

### Tool Landscape (2026)

| Tool | Strengths | Weaknesses |
|---|---|---|
| **GitHub Actions** | Free for public repos, huge marketplace, tight GitHub integration | Vendor-locked to GitHub |
| **Jenkins** | Self-hosted, ultimate flexibility, huge plugin ecosystem | Java-heavy, painful to maintain, dated UX |
| **GitLab CI** | Built into GitLab, similar feel to Actions | GitLab-only |
| **CircleCI** | Fast, good DX, Docker-native | Free tier limits |
| **Travis CI** | Pioneered cloud CI | In decline, mostly legacy |
| **ArgoCD** | Not strictly CI — GitOps CD for K8s | K8s-only (covered in Week 4) |
| **Tekton** | K8s-native pipelines | Steep learning curve |

**For CSOT: GitHub Actions** (primary) + Jenkins awareness (Module 7).

### Continuous Integration vs Delivery vs Deployment

```
Continuous Integration:
  push → build → test (every commit, automatically)

Continuous Delivery:
  push → build → test → ready to deploy (one-click)

Continuous Deployment:
  push → build → test → live (no human in the loop)
```

Most teams do **CI + Continuous Delivery**. Continuous Deployment requires very strong test coverage and feature flags.

---

## Module 2 — GitHub Actions Anatomy

### Where Workflows Live

```
.github/workflows/ci.yml
.github/workflows/release.yml
.github/workflows/deploy.yml
```

A repo can have multiple workflows. Each is a YAML file in `.github/workflows/`.

### Minimal Workflow

```yaml
name: CI

on: [push, pull_request]

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - name: Greet
        run: echo "Hello, CSOT!"
```

Push this file → go to the Actions tab → see it run.

Output in the run logs:
```
Run echo "Hello, CSOT!"
Hello, CSOT!
```

### Anatomy

```yaml
name: CI                            # Display name in the Actions tab

on:                                 # Triggers
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'             # daily at midnight
  workflow_dispatch:                # manual trigger button

jobs:
  build:                            # Job ID (any name)
    runs-on: ubuntu-latest          # Runner OS
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4   # marketplace action
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Install deps
        run: pip install -r requirements.txt
      - name: Run tests
        run: pytest -v
```

### Triggers — When Workflows Run

```yaml
on:
  push:
    branches: [main, develop]
    paths-ignore:
      - 'docs/**'
      - '*.md'

  pull_request:
    types: [opened, synchronize, reopened]

  release:
    types: [published]

  schedule:
    - cron: '0 3 * * *'             # 3 AM UTC daily

  workflow_dispatch:                # manual run button
    inputs:
      environment:
        description: 'Deploy to which env?'
        required: true
        type: choice
        options: [staging, prod]
```

### Runners

GitHub provides hosted runners:
- `ubuntu-latest` (most popular, fastest)
- `windows-latest`
- `macos-latest` (slower, more expensive)

Or you can self-host runners on your own hardware (free, but you maintain them).

**Free runner minutes** for public repos: **unlimited**.
For private repos: 2000 min/month free.

### The Marketplace

Every reusable building block is a published action. Find at [github.com/marketplace?type=actions](https://github.com/marketplace?type=actions).

Used as:
```yaml
- uses: actions/checkout@v4
- uses: docker/build-push-action@v5
- uses: trufflesecurity/trufflehog@main
```

**Security tip**: pin to a commit SHA for production, not a tag, since tags can be moved:

```yaml
- uses: actions/checkout@8f4b7f84864484a7bf31766abe9204da3cbe65b3   # v4
```

---

## Module 3 — Workflow Patterns

### Matrix Builds

Run the same job across multiple versions in parallel:

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest]
        node: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci && npm test
```

This produces **6 parallel jobs** (2 OS × 3 Node versions).

### Caching Dependencies

Without caching, every run reinstalls everything. Painful. Cache it:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'                    # automatic node_modules caching

# For Python:
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'
```

Custom cache:
```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/myapp
    key: ${{ runner.os }}-myapp-${{ hashFiles('**/lockfile') }}
    restore-keys: ${{ runner.os }}-myapp-
```

Typical impact: builds go from **5 min → 30 sec**.

### Secrets

Never hardcode. Add via GitHub UI: Settings → Secrets → Actions → New secret.

Use as:
```yaml
env:
  GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}

steps:
  - run: ./deploy.sh
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

**The `GITHUB_TOKEN`** is auto-injected by GitHub. Use it for things like pushing to GHCR, commenting on PRs, etc. No manual setup needed.

### Environments (with Protection Rules)

```yaml
jobs:
  deploy-prod:
    environment:
      name: production
      url: https://api.example.com
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

In repo settings → Environments → `production` → require:
- **Manual approval** by a reviewer
- **Wait timer** (5 min cool-down)
- **Branch restrictions** (only deploys from `main`)

### Composite Actions

Reusable steps bundled as a local action. Create `.github/actions/setup-env/action.yml`:

```yaml
name: Setup Environment
description: Checkout + setup language + install deps

runs:
  using: composite
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    - run: npm ci
      shell: bash
```

Use in workflows:
```yaml
steps:
  - uses: ./.github/actions/setup-env
  - run: npm test
```

### Reusable Workflows (`workflow_call`)

A whole workflow callable from other workflows. Create `.github/workflows/reusable-build.yml`:

```yaml
name: Reusable Build

on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: '20'
    secrets:
      NPM_TOKEN:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
        env:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      - run: npm run build
```

Call from `.github/workflows/ci.yml`:
```yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Job Dependencies (`needs:`)

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: echo lint

  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo test

  build:
    needs: [lint, test]              # only run if both pass
    runs-on: ubuntu-latest
    steps:
      - run: echo build
```

### Conditionals

```yaml
- name: Deploy only on main
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh

- name: Skip on docs-only PRs
  if: "!contains(github.event.head_commit.message, '[docs]')"
  run: pytest
```

---

## Module 4 — Quality Gates

The point of CI isn't just to "run something." It's to **prevent bad code from shipping**. Each gate must **fail the pipeline** on violation.

### Linting

**Python (Flake8):**
```yaml
- uses: actions/setup-python@v5
  with: { python-version: '3.12' }
- run: pip install flake8
- run: flake8 src/ --max-line-length=120
```

**JavaScript (ESLint):**
```yaml
- uses: actions/setup-node@v4
- run: npm ci
- run: npm run lint        # assumes package.json has "lint": "eslint ."
```

**Go (golangci-lint):**
```yaml
- uses: actions/setup-go@v5
  with: { go-version: '1.22' }
- uses: golangci/golangci-lint-action@v6
  with: { version: latest }
```

### Unit Testing with Coverage

```yaml
- run: pytest --cov=src --cov-report=xml --cov-fail-under=80
- uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: coverage.xml
```

The `--cov-fail-under=80` fails the build if coverage drops below 80%.

### JSON Schema Contract Testing

> The killer app: ensure your API's response shape never silently changes.

A FastAPI `/embed` endpoint returns:
```json
{
  "id": "abc-123",
  "embedding": [0.1, 0.2, ...],
  "model": "gemini-embedding-001"
}
```

Schema file `tests/embed.schema.json`:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["id", "embedding", "model"],
  "properties": {
    "id": {"type": "string"},
    "embedding": {
      "type": "array",
      "items": {"type": "number"},
      "minItems": 1
    },
    "model": {"type": "string"}
  },
  "additionalProperties": false
}
```

Test (`tests/test_embed_contract.py`):
```python
import json, requests, jsonschema, pathlib

def test_embed_response_matches_schema():
    schema = json.loads(pathlib.Path("tests/embed.schema.json").read_text())
    resp = requests.post("http://localhost:8000/embed", json={"text": "hello"})
    assert resp.status_code == 200
    jsonschema.validate(instance=resp.json(), schema=schema)
```

If a developer renames a field, removes one, or changes a type, this test fails *immediately* — before the change ships.

In the CI workflow:
```yaml
- name: Start app in background
  run: |
    pip install -r requirements.txt
    python -m src.server &
    sleep 3
- name: Schema contract test
  run: pytest tests/test_embed_contract.py -v
```

### Secret Scanning with TruffleHog

```yaml
- name: Secret scan
  uses: trufflesecurity/trufflehog@main
  with:
    path: ./
    base: ${{ github.event.repository.default_branch }}
    head: HEAD
    extra_args: --only-verified
```

This fails the build if any verified secret appears in the commits being added.

Alternative: **gitleaks**
```yaml
- uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Dependency Auditing

**Node:**
```yaml
- run: npm audit --audit-level=high
```

**Python:**
```yaml
- run: pip install pip-audit
- run: pip-audit -r requirements.txt --strict
```

**Go:**
```yaml
- run: go install golang.org/x/vuln/cmd/govulncheck@latest
- run: govulncheck ./...
```

### Dependabot (Free Dep Updates)

Create `.github/dependabot.yml`:
```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
```

GitHub will now open PRs to bump dependencies. Free, automatic.

### Branch Protection

In Repo Settings → Branches → Add rule:
- ✅ Require a pull request before merging
- ✅ Require status checks to pass:
  - `lint`
  - `test`
  - `schema-check`
  - `secret-scan`
- ✅ Require branches to be up to date before merging
- ✅ Require linear history (no merge commits — forces rebase)

Now `main` is **bulletproof**. No one can push directly; no one can merge red.

---

## Module 5 — Continuous Delivery: GHCR

### Registry Landscape

| Registry | Free Tier | Best For |
|---|---|---|
| **Docker Hub** | 1 private repo, rate limits on anonymous pulls | Public open-source |
| **GitHub Container Registry (GHCR)** | Unlimited public, free private (subject to GitHub plan) | **Default for this course** |
| **AWS ECR** | 500 MB private free | AWS workloads |
| **GCP Artifact Registry** | 0.5 GB free | GCP workloads |
| **Quay (RedHat)** | Free public | Enterprise / RedHat |

### Why GHCR is Our Default

- ✅ Free for public images, unlimited
- ✅ No separate account/login — uses your GitHub identity
- ✅ `GITHUB_TOKEN` auto-available in Actions (no secret to manage)
- ✅ Fast CDN
- ✅ Integrated with GitHub UI (Packages tab on your profile/repo)

### The Push-from-CI Workflow

`.github/workflows/release.yml`:

```yaml
name: Build & Push to GHCR

on:
  push:
    branches: [main]
    tags: ['v*']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}    # e.g., sumit-23/myapp

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write                     # !! required to push to GHCR

    steps:
      - uses: actions/checkout@v4

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=tag
            type=sha
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build & push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

After a push to `main`, your image appears at:
```
ghcr.io/sumit-23/myapp:latest
ghcr.io/sumit-23/myapp:main
ghcr.io/sumit-23/myapp:sha-a1b2c3d
```

### Making the Image Public

After the first push:
1. Go to your GitHub profile → Packages
2. Click your image → Package settings (right sidebar)
3. Change visibility → **Public**

Now anyone can `docker pull ghcr.io/sumit-23/myapp:latest` without auth.

### Pulling from GHCR

```bash
docker pull ghcr.io/sumit-23/myapp:latest

# For private images, log in first:
echo $GITHUB_PAT | docker login ghcr.io -u sumit-23 --password-stdin
```

---

## Module 6 — Repo Hygiene & Release Automation

### Semantic Versioning

```
MAJOR.MINOR.PATCH

1.0.0 → 1.0.1   bug fix
1.0.1 → 1.1.0   new feature, backwards compatible
1.1.0 → 2.0.0   breaking change
```

### Conventional Commits (Recap)

```
feat:     new feature                → MINOR bump
fix:      bug fix                    → PATCH bump
feat!:    feature with breaking      → MAJOR bump
docs:     documentation only         → no bump
chore:    maintenance                → no bump
refactor: code restructuring         → no bump
test:     test changes               → no bump
ci:       CI changes                 → no bump
perf:     performance improvement    → PATCH bump
```

### Automated Releases with `semantic-release`

`.github/workflows/release.yml`:
```yaml
name: Semantic Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0          # need full history for tag inspection

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

`release.config.js`:
```javascript
module.exports = {
  branches: ['main'],
  plugins: [
    '@semantic-release/commit-analyzer',
    '@semantic-release/release-notes-generator',
    '@semantic-release/changelog',
    '@semantic-release/github',
    ['@semantic-release/git', {
      assets: ['CHANGELOG.md'],
      message: 'chore(release): ${nextRelease.version}\n\n${nextRelease.notes}'
    }]
  ]
};
```

Now: merge a `feat:` commit → CI runs → version bumps to `1.1.0` → CHANGELOG updated → GitHub Release created → tag pushed → all automatically.

### Alternative: `release-please` (Google)

```yaml
- uses: googleapis/release-please-action@v4
  with:
    release-type: node
    token: ${{ secrets.GITHUB_TOKEN }}
```

Slightly different model — opens a "release PR" that you merge to cut a release.

### Required Status Checks

Once you have multiple workflows, mark which ones must pass:

Settings → Branches → main → require these checks:
- `lint`
- `test (ubuntu-latest, 20)`
- `schema-check`
- `secret-scan`

---

## Module 7 — Jenkins Awareness

> You will be asked about Jenkins in interviews. Read this section even if you never use Jenkins in CSOT.

### Why Jenkins Still Matters

- Released 2011. Battle-tested. Powers most enterprise CI/CD even in 2026.
- ~1700 plugins. Anything you can imagine, there's a plugin.
- Self-hosted → fits behind corporate firewalls
- Free, open source
- Most Indian DevOps job postings list Jenkins

### Architecture

```
[Jenkins Master]            ← brain: schedules, UI, plugin management
     │
     ├── [Agent 1] (Linux)   ← worker: actually runs the build
     ├── [Agent 2] (Windows) 
     └── [Agent 3] (Mac)
```

### Jenkinsfile Anatomy (Declarative Pipeline)

```groovy
pipeline {
    agent any

    environment {
        REGISTRY = 'ghcr.io/sumit-23/myapp'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Lint') {
            steps {
                sh 'pip install flake8'
                sh 'flake8 src/'
            }
        }
        stage('Test') {
            steps {
                sh 'pytest -v'
            }
        }
        stage('Build') {
            steps {
                sh "docker build -t ${REGISTRY}:${BUILD_NUMBER} ."
            }
        }
        stage('Push') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'ghcr',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login ghcr.io -u $USER --password-stdin"
                    sh "docker push ${REGISTRY}:${BUILD_NUMBER}"
                }
            }
        }
    }

    post {
        success { echo 'Pipeline OK' }
        failure { mail to: 'team@example.com', subject: 'Build failed' }
    }
}
```

### Actions vs Jenkins — When Each Wins

| Use Case | Pick |
|---|---|
| New public open-source project | GitHub Actions |
| You're already on GitHub | GitHub Actions |
| You need to run inside corporate network | Jenkins (or self-hosted Actions runners) |
| You want polyglot/legacy support | Jenkins (more plugins) |
| You want to manage no CI server | GitHub Actions / CircleCI |
| You're on GitLab | GitLab CI |

### Read but Don't Run

For CSOT, **do not install Jenkins** — it eats RAM and your time. Just read [the docs](https://www.jenkins.io/doc/) for 30 min. You'll recognize it in any future job interview.

---

## Module 8 — Other CI Tools (Brief)

### GitLab CI

Lives in `.gitlab-ci.yml` at repo root:

```yaml
stages: [lint, test, build]

lint:
  stage: lint
  image: python:3.12
  script:
    - pip install flake8
    - flake8 src/

test:
  stage: test
  image: python:3.12
  script:
    - pytest

build:
  stage: build
  image: docker:24
  services: [docker:24-dind]
  script:
    - docker build -t myapp .
  only: [main]
```

### CircleCI

`.circleci/config.yml`:

```yaml
version: 2.1
orbs:
  python: circleci/python@2.1
jobs:
  test:
    docker:
      - image: cimg/python:3.12
    steps:
      - checkout
      - python/install-packages:
          pkg-manager: pip
      - run: pytest
workflows:
  ci:
    jobs: [test]
```

### Travis CI

`.travis.yml`:

```yaml
language: python
python: '3.12'
install: pip install -r requirements.txt
script: pytest
```

(Travis is mostly legacy now. New projects pick Actions or GitLab CI.)

### Drone CI

Container-native, very lightweight. `.drone.yml`:

```yaml
kind: pipeline
type: docker
name: default
steps:
  - name: test
    image: python:3.12
    commands:
      - pip install -r requirements.txt
      - pytest
```

---

## Build 1 — PR Pipeline with 4 Quality Gates (Wed)

### Goal
A starter Node or Python repo with a GitHub Actions workflow that runs on every PR: **lint → test → JSON-schema → TruffleHog**. All four must pass for the PR to be mergeable (branch protection enforced).

### Step 1: Starter Repo

Use the provided starter at `csot-devops-2026/week-02-cicd-quality-registries/mini-project/starter-repo/`, or scratch-create:

```
my-week2-app/
├── src/
│   └── server.py
├── tests/
│   ├── test_unit.py
│   ├── test_schema.py
│   └── embed.schema.json
├── requirements.txt
├── .flake8
└── .github/
    └── workflows/
        └── ci.yml
```

`src/server.py` — a tiny FastAPI app:
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class EmbedRequest(BaseModel):
    text: str

@app.post("/embed")
def embed(req: EmbedRequest):
    # Pretend embedding (no real LLM call needed for tests)
    return {
        "id": "fake-1",
        "embedding": [0.1, 0.2, 0.3],
        "model": "demo-v1"
    }
```

`tests/embed.schema.json` — as in [Module 4](#json-schema-contract-testing).

`tests/test_schema.py`:
```python
from fastapi.testclient import TestClient
import json, jsonschema, pathlib
from src.server import app

client = TestClient(app)

def test_embed_schema():
    schema = json.loads(pathlib.Path("tests/embed.schema.json").read_text())
    r = client.post("/embed", json={"text": "hello"})
    assert r.status_code == 200
    jsonschema.validate(instance=r.json(), schema=schema)
```

`requirements.txt`:
```
fastapi==0.115.0
uvicorn==0.30.0
pydantic==2.9.0
pytest==8.3.0
httpx==0.27.0
jsonschema==4.23.0
flake8==7.1.0
```

### Step 2: The CI Workflow

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.12', cache: 'pip' }
      - run: pip install -r requirements.txt
      - run: flake8 src/ tests/ --max-line-length=120

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.12', cache: 'pip' }
      - run: pip install -r requirements.txt
      - run: pytest tests/test_unit.py -v

  schema-check:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.12', cache: 'pip' }
      - run: pip install -r requirements.txt
      - run: pytest tests/test_schema.py -v

  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - name: TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
          extra_args: --only-verified
```

### Step 3: Branch Protection

Settings → Branches → Add rule for `main`:
- ✅ Require PR before merging
- ✅ Require status checks to pass: `lint`, `test`, `schema-check`, `secret-scan`
- ✅ Require branches to be up to date

### Step 4: Verify Each Gate Breaks the Pipeline

Make 4 PRs (in 4 branches) deliberately breaking each gate:
1. **Bad lint**: add `import os, sys, json` on a single line → flake8 fails
2. **Broken test**: change return value in `server.py` → unit test fails
3. **Schema break**: rename `embedding` → `embeddings` → schema test fails
4. **Leaked secret**: commit `AWS_KEY=AKIAIOSFODNN7EXAMPLE` → TruffleHog fails

Each PR should show a red X on the failing check.

### ✅ Build 1 Complete When:
- All 4 jobs run on every PR
- Each gate can be made to fail individually (4 screenshots)
- Branch protection prevents merging on red

---

## Build 2 — Build & Push to GHCR (Fri)

### Goal
Extend Build 1: on push to `main`, build a Docker image, push to GHCR tagged with `latest` and the commit SHA, but **skip the push entirely if any quality gate fails**.

### Step 1: Dockerfile

Add `Dockerfile` at repo root:
```dockerfile
FROM python:3.12-slim AS base
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ ./src/
EXPOSE 8000
CMD ["uvicorn", "src.server:app", "--host", "0.0.0.0", "--port", "8000"]
```

(Multi-stage build details come in Week 3 — for now this is fine.)

### Step 2: Add a Reusable Build Workflow

`.github/workflows/reusable-build-push.yml`:
```yaml
name: Build & Push (Reusable)

on:
  workflow_call:
    inputs:
      image-tag-prefix:
        type: string
        default: ''

jobs:
  build-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=raw,value=latest,enable={{is_default_branch}}
            type=sha,prefix=sha-
            type=ref,event=tag

      - name: Set up Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build & push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Step 3: Wire Into CI with `needs:`

Update `.github/workflows/ci.yml`:

```yaml
# (keep lint, test, schema-check, secret-scan as before)

  build-push:
    needs: [lint, test, schema-check, secret-scan]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    uses: ./.github/workflows/reusable-build-push.yml
```

### Step 4: Verify

Push to `main`:
1. All 4 quality jobs run
2. `build-push` runs only after all 4 pass
3. Image appears at `ghcr.io/<user>/<repo>:latest` and `:sha-<short>`

Verify:
```bash
docker pull ghcr.io/sumit-23/my-week2-app:latest
docker run -p 8000:8000 ghcr.io/sumit-23/my-week2-app:latest
curl -X POST http://localhost:8000/embed -H "Content-Type: application/json" -d '{"text":"hi"}'
# {"id":"fake-1","embedding":[0.1,0.2,0.3],"model":"demo-v1"}
```

### Step 5: Demonstrate "Bad Code Doesn't Ship"

Break a unit test on `main` (push directly bypassing protection just for the demo, or PR-and-merge-rapidly):
- `lint`/`test`/`schema-check`/`secret-scan` run
- One fails
- `build-push` **does NOT run** (because of `needs:`)
- No new image appears in GHCR

Screenshot the Actions tab showing the skipped `build-push` job.

### ✅ Build 2 Complete When:
- Image appears in GHCR after successful main push
- Image is pullable + runnable
- Failing quality gate prevents `build-push` from running (screenshot)
- Image is tagged with both `latest` and `sha-<short>`

---

## Weekly Mini-Project — Production-Grade CI Pipeline

> **Submission deadline: Sunday 11:59 PM (IST)**

### 🟢 Part A — Local Track (MANDATORY, 50 pts)

This entire week's mini-project lives in **GitHub's free cloud** (Actions runners + GHCR). **No AWS, no own server, no credit card needed.**

Take a sample app (Node, Python, or Go — your choice) and build a complete CI/CD pipeline with **9 features**.

#### Required Pipeline Features

| # | Feature | Tool |
|---|---|---|
| 1 | **Lint** stage | ESLint / Flake8 / golangci-lint |
| 2 | **Test** stage + coverage artifact | Jest / pytest / go test |
| 3 | **JSON-schema contract test** — fails on API response shape change | `jsonschema` / `ajv` |
| 4 | **Secret scan** — full history | TruffleHog or gitleaks |
| 5 | **Dependency scan** | `npm audit` / `pip-audit` / `govulncheck` |
| 6 | **Build & push** to GHCR with `latest` + commit SHA tags | `docker/build-push-action@v5` |
| 7 | **Auto-tag releases** on merge to `main` | `semantic-release` or `release-please` |
| 8 | **Matrix build** — at least one job on ≥ 2 language versions | matrix strategy |
| 9 | **File-size guard** — fail if any file > 1 MB | custom step |

#### Sample File-Size Guard

```yaml
- name: File size check
  run: |
    big=$(find . -type f -size +1M -not -path './.git/*' -not -path './node_modules/*')
    if [ -n "$big" ]; then
      echo "::error::Files larger than 1MB found:"
      echo "$big"
      exit 1
    fi
```

#### Submission Requirements (Part A)

- ✅ Public GitHub repo with all 9 features
- ✅ Branch protection on `main` requiring all gates to pass
- ✅ README with what each stage does, screenshots of green pipeline
- ✅ **At least one reusable workflow** (`workflow_call`) or composite action
- ✅ All secrets in GitHub Secrets (none hardcoded)
- ✅ **One screenshot showing a failing run** (proves your guards work)
- ✅ Image published to your GHCR namespace, **public**

#### Grading Rubric — Part A (50 pts)

| Criterion | Points |
|---|---|
| All 9 pipeline features working | 25 |
| GHCR push works, image is public, correct tags | 5 |
| JSON-schema gate breaks pipeline correctly (failure screenshot) | 5 |
| TruffleHog gate breaks pipeline correctly (failure screenshot) | 5 |
| Reusable workflow / composite action | 5 |
| Documentation & screenshots | 5 |

### 🟡 Part B — Cloud Track (OPTIONAL BONUS, +10 pts)

Add a **deploy** stage that ships the GHCR image to a real running machine after `main` is updated. Pick **one** path:

#### Path 1 — VPS/Cloud VM (SSH + Docker)

Use your Week 1 Part B cloud VM (or provision a new one). Add this job:

```yaml
deploy:
  needs: build-push
  if: github.ref == 'refs/heads/main'
  runs-on: ubuntu-latest
  environment:
    name: production
    url: https://api.yourdomain.com
  steps:
    - name: Deploy via SSH
      uses: appleboy/ssh-action@v1
      with:
        host: ${{ secrets.SSH_HOST }}
        username: ${{ secrets.SSH_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          docker pull ghcr.io/${{ github.repository }}:sha-${{ github.sha }}
          docker stop myapp || true
          docker rm myapp || true
          docker run -d --name myapp -p 8000:8000 \
            --restart unless-stopped \
            ghcr.io/${{ github.repository }}:sha-${{ github.sha }}
```

#### Path 2 — Cloudflare Pages (Static Sites Only)

If your project is a static site (Hugo / Astro / Next.js static export):
```yaml
- uses: cloudflare/pages-action@v1
  with:
    apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    projectName: myapp
    directory: dist
    gitHubToken: ${{ secrets.GITHUB_TOKEN }}
```

#### Path 3 — Fly.io / Render Free Tier

```yaml
- uses: superfly/flyctl-actions/setup-flyctl@master
- run: flyctl deploy --remote-only
  env:
    FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

#### Submission Requirements (Part B)

- ✅ Deploy job runs on merge to `main`
- ✅ Image pinned by **commit SHA** (not `latest`) for traceability
- ✅ GitHub **Environment** (`production`) with required reviewer (manual approval)
- ✅ Public URL responds after deploy

#### Grading Rubric — Part B (+10 pts)

| Criterion | Points |
|---|---|
| Deploy step actually updates the running app | 4 |
| Image pinned by commit SHA | 2 |
| GitHub Environment with manual approval | 2 |
| Public URL works after deploy | 2 |

---

## Alternative Mini-Project Ideas

### 1. Auto-Deploying Personal Site
Build a static site (Hugo / Astro / Next.js / Vite). Full CI/CD: push to `main` → build → deploy to **Cloudflare Pages**. **Preview deploys** for every PR (Cloudflare Pages does this automatically; configure via Wrangler). Add a Lighthouse score gate that fails on score < 90.

### 2. PR Bot / Code Review Assistant
A GitHub Action that auto-comments on every PR with:
- Lint results (diff format)
- Code coverage diff vs main
- File-size delta
- Opens a fix-PR if formatting is off (via `peter-evans/create-pull-request`)

### 3. Release-Notes Generator
Action triggered on tag push that:
- Reads conventional commits between this and the previous tag
- Generates a Markdown CHANGELOG
- Posts to Discord/Slack via webhook
- Creates a GitHub Release with the notes

### 4. Multi-Repo CI Status Dashboard
A reusable workflow + a static dashboard (deployed via Cloudflare Pages) showing CI status across 5+ of your own repos. Updates hourly via a scheduled Action.

### 5. CI Pipeline Template Pack
3 polished reusable workflows (Node, Python, Go). Published on GitHub Marketplace with proper README, version tagging, and at least 3 example consumer repos. Goal: someone *actually adopts* one.

### 6. (AI Track) LLM Endpoint Eval Pipeline
CI calls a FastAPI + Gemini endpoint and asserts the response matches a strict JSON schema. Pipeline fails on schema break. Add a second test: latency must be < 3s p95 over 10 runs (skip flaky LLM tests if rate-limited).

---

## Weekly Quiz Topics

- Pipeline stages and their order
- GitHub Actions YAML syntax: workflows, jobs, steps
- When to use matrix builds
- Caching strategies for speed
- Secrets management; what `GITHUB_TOKEN` is
- Reusable workflows vs composite actions
- GHCR vs Docker Hub vs ECR
- Conventional commits + semantic-release flow
- Branch protection rules
- Why JSON-schema contract testing matters
- Jenkins architecture (master/agent), Jenkinsfile basics

---

## Weekly Contest — Green Build Race

> **Released Saturday 9 AM, due Sunday 11:59 PM**

You'll receive a starter repo with **5 deliberate pipeline issues**. Your tasks:

1. **Fix all 5** to get the pipeline green
2. **Add 3 new stages**: security scan, build, artifact upload
3. **Reduce total runtime** by ≥ 40% using caching + parallelization

**Judging:**
- Pipeline correctness (5 pts)
- Total runtime (3 pts — leaderboard, faster = higher)
- Use of reusable workflows / composite actions (2 pts)

Sample bugs (you won't know which):
- `runs-on: ubntu-latest` (typo)
- Missing `actions/checkout@v4`
- Secrets passed as inputs without `secrets:` block
- `needs:` job referencing a job that doesn't exist
- Matrix syntax error (`fail-fast: True` instead of `true`)

---

## Resources

### Official Docs
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [GHCR Docs](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [12-Factor App](https://12factor.net/)

### Books
- *Continuous Delivery* — Jez Humble & David Farley (Ch 1, 5)
- *Accelerate* — Forsgren, Humble, Kim (DORA metrics — what good CI/CD predicts)

### Videos
- TechWorld with Nana — GitHub Actions tutorial
- TechWorld with Nana — Jenkins crash course (first 45 min)
- DevOps Toolkit — GitHub Actions deep dive

### Tools to Bookmark
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [act](https://github.com/nektos/act) — run Actions locally
- [actionlint](https://github.com/rhysd/actionlint) — lint your workflows
- [TruffleHog](https://github.com/trufflesecurity/trufflehog)

### Reference Workflows
- [Docker official sample workflows](https://github.com/docker/build-push-action)
- [actions/starter-workflows](https://github.com/actions/starter-workflows)

---

**Next week: Week 4 — Kubernetes, Helm & GitOps.** Deploy the GHCR image you built this week to a local Kind cluster with Helm and ArgoCD.
