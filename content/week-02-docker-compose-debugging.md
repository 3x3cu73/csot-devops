# Week 2 — Docker, Compose & Container Debugging

> **Theme**: Package once, run anywhere — then debug it when it breaks (because it will).
>
> **Time Budget**: ~10 hours (reading 4h + builds 2h + contest 3h + mini-project overlap)

This file is **Week 2 of the CSOT DevOps weekly curriculum**. Week 1 gave you Linux, shell, and networking. Week 2 gives you **containers** — the skill every later week (CI, Kubernetes, cloud) builds on.

**How to use this guide:** read Modules 0–7 to learn Docker. Read **Module 8** before attempting contest tasks 03–12. Hard contest tasks require downloading a broken fixture and debugging locally for 60–120 minutes each.

---

## Table of Contents

1. [Tools & Software Used](#tools--software-used)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Learning Outcomes](#learning-outcomes)
4. **[Week 2 Contest — mandatory (200 pts)](#week-2-contest--mandatory-200-points)**
5. [Module 0 — Why Containers Exist](#module-0--why-containers-exist)
6. [Module 1 — Docker CLI, Images, Containers](#module-1--docker-cli-images-containers)
7. [Module 2 — Dockerfiles and Build Context](#module-2--dockerfiles-and-build-context)
8. [Module 3 — Docker Debugging Workflow](#module-3--docker-debugging-workflow)
9. [Module 4 — Compose, Networking, and Volumes](#module-4--compose-networking-and-volumes)
10. [Module 5 — Image Hygiene, Multi-Stage Builds, Non-Root](#module-5--image-hygiene-multi-stage-builds-non-root)
11. [Module 6 — Healthchecks, Logs, and 12-Factor Runtime](#module-6--healthchecks-logs-and-12-factor-runtime)
12. [Module 7 — Basic Image Security with Trivy](#module-7--basic-image-security-with-trivy)
13. **[Module 8 — Incident Response Playbooks](#module-8--incident-response-playbooks-contest-prep)** ← contest prep
14. [Self-Study Pace](#self-study-pace)
15. [Build 1 — Containerize a Single App](#build-1--containerize-a-single-app)
16. [Build 2 — Multi-Service Compose Stack](#build-2--multi-service-compose-stack)
17. [Weekly Mini-Project — Dockerized App Stack](#weekly-mini-project--dockerized-app-stack)
18. [Alternative Mini-Project Ideas](#alternative-mini-project-ideas)
19. [Weekly Quiz Topics](#weekly-quiz-topics)
20. [Contest reference — tasks, scoring, rules](#contest-reference--tasks-scoring-rules)
21. [Resources](#resources)

---

## Tools & Software Used

> **Mac & Windows users:** run Docker **inside Ubuntu** (WSL2 or multipass), same as Week 1. See [Prerequisites](#prerequisites--setup).

| Tool | Purpose | Install (Linux / WSL2 / multipass) |
|------|---------|-----------------------------------|
| **Docker Engine** | Container runtime | `curl -fsSL https://get.docker.com \| sh` then `sudo usermod -aG docker $USER` |
| **docker compose** | Multi-container stacks | Plugin: `sudo apt install docker-compose-plugin` |
| **jq** | Parse `docker inspect` JSON | `sudo apt install jq` |
| **curl** | HTTP probes | Pre-installed |
| **git** | Clone fixtures / mini-project | `sudo apt install git` |
| **Trivy** | Image vulnerability scan | See [Module 7](#module-7--basic-image-security-with-trivy) |
| **nc / netcat-openbsd** | Test TCP connectivity inside containers | `sudo apt install netcat-openbsd` |

**Verification:**

```bash
docker run hello-world
docker compose version
jq --version
for tool in docker git curl jq; do
  command -v "$tool" >/dev/null && echo "✓ $tool" || echo "✗ $tool MISSING"
done
```

Expected: `Hello from Docker!` and compose plugin version ≥ 2.x.

**Disk:** allow ≥ 10 GB free for images and contest fixtures.

---

## Prerequisites & Setup

- **Week 1 complete** — comfortable with shell, env vars, `curl`, basic networking
- **Linux / WSL2 / multipass** — same setup as Week 1 (`week-01-linux-networking-git.md`)
- **Docker daemon running** — `docker info` must succeed
- **User in `docker` group** — or prefix commands with `sudo docker` consistently
- **csot CLI** — from Week 1 contest (`curl -fsSL https://csot-devops.devclub.in/install.sh | bash`)

### Install Docker (Linux / inside WSL2 / multipass)

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker "$USER"
newgrp docker    # or log out and back in
docker run hello-world
```

### macOS reminder

Use `multipass shell dev` for every command in this guide — Docker inside the Ubuntu VM, not on macOS host.

---

## Learning Outcomes

By the end of this week, you can:

- Explain image vs container vs registry vs daemon
- Build and run images with `docker build`, `docker run`, `docker ps`, `docker logs`, `docker exec`, `docker inspect`
- Write a correct Dockerfile for a Python app with `.dockerignore`
- Run a multi-service stack with `docker compose` — app + Postgres + Redis
- Connect services by **service name**, not `localhost`
- Use named volumes so database data survives container recreation
- Debug multi-bug broken stacks using logs, inspect, exec, and HTTP probes
- Apply non-root users, multi-stage builds, and healthchecks
- Scan images with Trivy and read HIGH/CRITICAL findings
- Complete contest incident tasks by reproducing failures locally (not paste-only fixes)

---

## Week 2 Contest — mandatory (200 points)

> **Mandatory for every Week 2 participant.** Runs all week on **[csot-devops.devclub.in](https://csot-devops.devclub.in)**.
> Contest name: **Docker Rescue Sprint**.

| | |
|---|---|
| **Platform** | [csot-devops.devclub.in/contest/week-02](https://csot-devops.devclub.in/contest/week-02) |
| **Points** | **200 total** — ~22 pts warm-ups, ~178 pts multi-step incident debugging |
| **How** | `csot submit ./solutions` or `csot submit ./solutions -t 07` |
| **Fixtures** | Hard tasks (03–12) ship broken stack tarballs — **must run locally** |
| **Time penalty** | Live VM tasks lose points the longer the sandbox runs (max −20%/task). [Details](#live-sandbox-time-penalty-the-2-am-page-clock-) |

```bash
curl -fsSL https://csot-devops.devclub.in/install.sh | bash
csot login
csot submit ./my-solutions
csot submit ./my-solutions -t 03
```

**How to solve hard tasks:** read **Module 8 — Incident Response Playbooks** in this doc. Each task has a step-by-step debugging sequence, bug checklist, and verify-before-submit commands.

Full task list and rules: **[Contest reference](#contest-reference--tasks-scoring-rules)** at the end of this doc.

---

## Self-Study Pace

| When | Read | Do | Contest |
|------|------|-----|---------|
| Early week | Modules 0–2 | Install Docker, [Build 1](#build-1--containerize-a-single-app) | 01, 02 |
| Mid week | Modules 3–4, 8 | [Build 2](#build-2--multi-service-compose-stack) | 03, 04 |
| Late week | Modules 5–7, 8 playbooks | Practice fixtures | 05–09 |
| Before Sunday | Module 8 (tasks 11–12) | Full stack locally | 10–12 |
| Sunday | — | Mini-project repo | final submits |

**Rule:** tasks 03–12 need the fixture tarball running on your machine. The task page shows symptoms; Module 8 shows how to debug.

---

## Module 0 — Why Containers Exist

### The problem containers solve

Your app works on your laptop. It fails on the server. Different Python version, missing package, wrong env var, port already in use, path differences — "works on my machine" is the default state of software.

A **container** packages your app with its runtime dependencies into an **image** — a frozen template you can run identically anywhere Docker runs.

### Container ≠ VM

A container is an **isolated process** on the host kernel (namespaces + cgroups). It shares the host kernel. A VM runs a full guest OS.

```
Containers: fast start, low overhead, shared kernel
VMs:        slow start, high overhead, strong isolation
```

**Rule of thumb:** containers for apps; VMs for OSes.

### Images vs containers

- **Image** = read-only template (the recipe)
- **Container** = running instance of an image (the meal)

```bash
docker pull nginx:1.27
docker run -d --name web nginx:1.27
docker ps
docker stop web && docker rm web
```

### Where Docker fits in the program

```
Week 1: Linux + networking (the OS)
Week 2: Docker + Compose (package the app)     ← you are here
Week 3: CI/CD + GHCR (automate the build)
Week 4: Kubernetes (orchestrate containers)
Week 5: Cloud + observability + capstone
```

---

## Module 1 — Docker CLI, Images, Containers

### Daily commands

```bash
docker pull nginx:1.27
docker run -d --name web -p 8080:80 nginx:1.27
docker ps
docker ps -a                    # includes stopped
docker logs web
docker logs -f --tail 50 web    # follow
docker exec -it web sh          # shell inside
docker inspect web | jq '.[0].State'
docker port web
docker stop web
docker rm web
docker images
docker rmi nginx:1.27
```

### Port publishing

```bash
docker run -d -p 8080:80 nginx    # host:container
curl http://localhost:8080/
```

`-p 8080:80` maps **host port 8080** to **container port 80**. `EXPOSE` in a Dockerfile is documentation only — it does not publish ports.

### When things go wrong

| Symptom | First commands |
|---------|----------------|
| Container exits immediately | `docker ps -a` → `docker logs <name>` |
| Port unreachable from host | `docker port <name>` → check `-p` mapping |
| Command not found inside | `docker exec <name> which <cmd>` — container has its own filesystem |

### Try it yourself

```bash
docker run -d --name nginx-demo -p 8080:80 nginx:1.27
curl -I http://localhost:8080/
docker exec nginx-demo cat /etc/nginx/nginx.conf | head -5
docker stop nginx-demo && docker rm nginx-demo
```

---

## Module 2 — Dockerfiles and Build Context

### Minimal Python app

`app.py`:

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json, os, datetime

class H(BaseHTTPRequestHandler):
    def do_GET(self):
        body = json.dumps({
            "ok": True,
            "service": "csot-week2",
            "time": datetime.datetime.now().isoformat(),
            "path": self.path,
        }).encode()
        self.send_response(200)
        self.send_header("Content-Type", "application/json")
        self.end_headers()
        self.wfile.write(body)
    def log_message(self, fmt, *args):
        print(fmt % args, flush=True)

if __name__ == "__main__":
    host = os.environ.get("HOST", "0.0.0.0")
    port = int(os.environ.get("PORT", "8000"))
    print(f"listening on {host}:{port}", flush=True)
    HTTPServer((host, port), H).serve_forever()
```

`Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY app.py .

CMD ["python", "app.py"]
```

Build and run:

```bash
docker build -t csot-hello .
docker run --rm -p 8000:8000 csot-hello
curl http://localhost:8000/
```

### Dockerfile instructions you need

| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image |
| `WORKDIR` | Working directory inside image |
| `COPY` | Copy files from build context |
| `RUN` | Run command at **build** time (new layer) |
| `ENV` | Environment variable (runtime) |
| `EXPOSE` | Document which port app uses |
| `CMD` | Default command (overridable) |
| `ENTRYPOINT` | Main executable |
| `USER` | Run as non-root |
| `HEALTHCHECK` | Docker probes container health |

### Layer caching — order matters

```dockerfile
# BAD — code change reinstalls all deps
COPY . .
RUN pip install -r requirements.txt

# GOOD — deps cached unless requirements.txt changes
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
```

### `.dockerignore`

```
.git
.env
.env.*
__pycache__
*.pyc
.venv
node_modules
*.log
.DS_Store
```

Without this, `COPY . .` bakes secrets and caches into the image.

### Common Dockerfile mistakes (contest tasks exploit these)

| Mistake | Symptom |
|---------|---------|
| App binds `127.0.0.1` | Container "running" but `curl` from host fails |
| Shell-form `CMD python app.py` | SIGTERM not forwarded; slow shutdown |
| `COPY . .` without `.dockerignore` | Huge image; secrets in layers |
| `USER app` without `chown` | `Permission denied` on write paths |

---

## Module 3 — Docker Debugging Workflow

> This module is the core skill for contest tasks 03–12. Memorizing fixes does not work — you must **observe** the running system.

### The incident loop

```text
1. REPRODUCE   — make the failure happen reliably
2. OBSERVE     — ps, logs, inspect, exec, curl
3. HYPOTHESIZE — simplest explanation fitting ALL observations
4. FIX ONE     — smallest change
5. RE-TEST     — full test, not just the line you changed
6. REPEAT      — next bug often appears after the first fix
7. DOCUMENT    — symptom → command → root cause → fix
```

### Toolkit

```bash
docker compose ps
docker compose logs --tail=50 app db
docker compose config
docker logs <name> --tail=100
docker inspect <name> | jq '.[0].State, .[0].NetworkSettings.Networks'
docker exec -it <name> sh
docker exec <name> id
docker exec <name> env | sort
docker exec <name> ss -lntp 2>/dev/null || netstat -lntp
docker port <name>
curl -sv http://localhost:8000/health
curl -sv http://localhost:8000/api/users
```

### Bind address trap

Inside a container, `127.0.0.1` means **the container itself**, not the host and not other containers.

```bash
# App listening on 127.0.0.1:8000 — WRONG for Docker
docker exec app ss -lntp
# LISTEN 127.0.0.1:8000

# Fix: HOST=0.0.0.0 so host port mapping works
```

### Don't trust the first log line

| Log says | Often actually means |
|----------|----------------------|
| `redis connection refused` | Wrong hostname, wrong network, or Redis not ready — verify with `nc` |
| `connection refused 127.0.0.1:5432` | Using localhost instead of `db` service name |
| `Permission denied` | USER without chown, or volume owned by root |
| `/health` returns 200 but API 500 | Shallow healthcheck; DB/config still broken |

### Incident note template

```text
Symptom:
Command used:
Important output:
Root cause:
Fix:
How to prevent:
```

---

## Module 4 — Compose, Networking, and Volumes

### Why Compose exists

Real apps need app + database + cache + workers. Compose defines them in one `compose.yaml` file.

### Example stack

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgres://app:secret@db:5432/appdb
      REDIS_HOST: redis
      HOST: "0.0.0.0"
      PORT: "8000"
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d appdb"]
      interval: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s

volumes:
  pgdata:
```

### Service-name DNS

Inside the Compose network, `db` resolves to the Postgres container. **`localhost` does not.**

```bash
docker exec app getent hosts db
docker exec app nc -zv db 5432
```

### `depends_on` is not readiness

Without `condition: service_healthy`, Compose only waits for the container to **start**, not for Postgres to **accept connections**. Result: flaky first request or crash loop.

### Volumes

| Type | Use |
|------|-----|
| **Named volume** | Persistent DB data — survives `docker compose down` |
| **Bind mount** | Dev code reload — `./src:/app/src` |
| **tmpfs** | Ephemeral scratch |

```bash
docker compose down       # stops containers, KEEPS volumes
docker compose down -v    # DESTROYS volumes and data
```

### Postgres volume path trap

The official Postgres image stores data at:

```text
/var/lib/postgresql/data    ← correct
/var/lib/postgresql         ← WRONG (contest task 08)
```

### Lifecycle commands

```bash
docker compose up -d --build
docker compose ps
docker compose logs -f app
docker compose exec app sh
docker compose exec db psql -U app -d appdb
docker compose config
docker compose down
docker compose down -v
```

---

## Module 5 — Image Hygiene, Multi-Stage Builds, Non-Root

### Multi-stage build (Python)

```dockerfile
FROM python:3.12 AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

FROM python:3.12-slim AS runtime
WORKDIR /app
COPY --from=builder /wheels /wheels
COPY requirements.txt .
RUN pip install --no-cache-dir --no-index --find-links=/wheels -r requirements.txt && rm -rf /wheels
COPY app.py .
RUN useradd -m -u 10001 appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8000
CMD ["python", "app.py"]
```

Compare sizes:

```bash
docker images --format '{{.Repository}}:{{.Tag}} {{.Size}}'
```

### Non-root runtime

```dockerfile
RUN adduser --disabled-password --gecos "" appuser
COPY --chown=appuser:appuser . .
RUN mkdir -p /app/data && chown appuser:appuser /app/data
USER appuser
```

If you add `USER` without `chown`, writes to `/app/data` fail with `Permission denied`.

### Inspect layers

```bash
docker history myapp:latest
docker image inspect myapp:latest | jq '.[0].Size, .[0].Config.User'
```

---

## Module 6 — Healthchecks, Logs, and 12-Factor Runtime

### 12-factor in containers

| Factor | Docker practice |
|--------|-----------------|
| **Config** | Env vars via `environment:` or `--env-file` |
| **Logs** | Write to stdout/stderr — `docker logs` captures them |
| **Processes** | Stateless app containers; state in DB/volumes |
| **Disposability** | Fast start; handle SIGTERM gracefully |

### Healthcheck in Dockerfile

```dockerfile
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health')" || exit 1
```

### Healthcheck in compose

```yaml
app:
  healthcheck:
    test: ["CMD", "curl", "-fsS", "http://localhost:8000/health"]
    interval: 10s
    timeout: 3s
    retries: 3
```

### Shallow healthcheck trap

A `/health` endpoint that returns 200 without checking DB is useless for readiness. Always test the real API:

```bash
curl http://localhost:8000/health      # may pass
curl http://localhost:8000/api/users   # may 500 — real problem
```

---

## Module 7 — Basic Image Security with Trivy

### Install Trivy (Linux)

```bash
sudo apt-get install -y wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update && sudo apt install -y trivy
```

### Scan images

```bash
trivy image python:3.12
trivy image python:3.12-slim
trivy image --severity HIGH,CRITICAL myapp:latest
```

Most findings come from the **base image**. Switching `python:3.12` → `python:3.12-slim` often drops findings dramatically.

**Contest task 06** asks you to shrink images — Trivy helps you see what you are shipping.

---

## Module 8 — Incident Response Playbooks (Contest Prep)

> Read this module **after** Modules 1–4, then revisit as you attempt each contest task.
> Hard tasks are **not** solvable from the task page alone — download the fixture, run it, explore, fix, re-test.

### The debugging loop (use on every hard task)

```text
1. REPRODUCE   — make the failure happen reliably
2. OBSERVE     — ps, logs, inspect, exec, curl (don't trust one log line)
3. HYPOTHESIZE — pick the simplest explanation that fits all observations
4. FIX ONE     — smallest change; re-test
5. REPEAT      — partial fixes often reveal the next bug
6. DOCUMENT    — symptom → command → root cause → fix (for NOTES.md)
```

### Universal toolkit

```bash
# Stack state
docker compose ps
docker compose logs --tail=50 app db redis worker
docker compose config                    # merged config — read this carefully

# Single container
docker logs <name> --tail=100
docker inspect <name> | jq '.[0].State, .[0].NetworkSettings, .[0].Config.Env'
docker exec -it <name> sh
docker exec <name> id
docker exec <name> ls -la /app /app/data
docker port <name>

# Networking (from inside app container)
docker exec app getent hosts db redis
docker exec app sh -c 'nc -zv db 5432; nc -zv redis 6379'
docker exec app sh -c 'ss -lntp || netstat -lntp'

# HTTP probes (from host)
curl -sv http://localhost:8000/health
curl -sv http://localhost:8000/api/users
for i in $(seq 1 20); do curl -so /dev/null -w "%{http_code}\n" http://localhost:8000/api/users; done

# Volumes
docker volume ls
docker exec db ls -la /var/lib/postgresql/data

# Cleanup between attempts
docker compose down          # keeps volumes
docker compose down -v       # wipes volumes — only when you mean it
```

### Don't trust the first error line

| Log says | Often actually means |
|----------|----------------------|
| `connection refused` to `redis` | Wrong hostname, wrong network, or service not up — verify with `getent hosts` + `nc` |
| `connection refused` to `127.0.0.1` | Using localhost inside a container — need service name |
| `Permission denied` on `/app/data` | USER change without chown, or volume mounted with root ownership |
| `healthy` on `/health` but API 500 | Shallow healthcheck; DB/Redis/config still broken |
| `Restarting (1)` | Crash loop — check *first* error in logs, not the latest flood |

---

### Contest ↔ Module map

| Task | Title | Read first | Est. time |
|------|-------|------------|-----------|
| 01 | Hello Container | Module 2 | 30 min |
| 02 | Context Leak | Module 2, 5 | 30 min |
| 03 | The Flapping Stack | Module 3, 4, 6 | 60–90 min |
| 04 | Misleading Redis Error | Module 3, 4 | 60 min |
| 05 | Migration Before Ready | Module 4, 6 | 75 min |
| 06 | Slim the Image | Module 5, 7 | 45 min |
| 07 | Permission Rabbit Hole | Module 2, 5 | 60–90 min |
| 08 | Data Ghost Volume | Module 4 | 60 min |
| 09 | Exec Rescue (5 bugs) | Module 3, 8 | 60–90 min |
| 10 | Container Doctor | Module 1, 3 | 45 min |
| 11 | SSH: 2 AM Incident | Module 3, 4, 6, 8 | 90–120 min |
| 12 | Full Stack Rescue | All modules | 90–120 min |

---

### Playbook: Task 01 — Hello Container (5 pts)

**Goal:** Write `01/Dockerfile` for a provided `server.py`.

**Steps:**
1. Read `server.py` — what port? what deps?
2. `FROM python:3.12-slim`
3. `WORKDIR /app`
4. Copy `requirements.txt` first, `RUN pip install`, then `COPY` app code (cache-friendly).
5. `CMD ["python", "server.py"]` — exec form, not shell form.
6. Test locally:

```bash
docker build -t task01 .
docker run --rm -p 8000:8000 task01
curl http://localhost:8000/
```

**Pass criteria:** build succeeds, `GET /` returns expected JSON, slim base image.

---

### Playbook: Task 02 — Context Leak (5 pts)

**Goal:** Write `02/.dockerignore` so secrets and caches never enter the build context.

**Steps:**
1. List the fixture tree: `find . -type f | head -50`
2. Exclude: `.env`, `.git`, `node_modules`, `.venv`, `__pycache__`, `*.log`, `.DS_Store`
3. **Do not** exclude: `app.py`, `requirements.txt`, `package-lock.json`
4. Verify by building and checking image contents:

```bash
docker build -t task02 .
docker run --rm task02 find /app -type f
```

**Pass criteria:** secrets/caches absent; source files present.

---

### Playbook: Task 03 — The Flapping Stack (10 pts) ★★★

**Symptom:** `/health` always 200, `/api/users` fails ~30% of the time. Restarting `app` helps briefly.

**Submit:** `03/compose.yaml` + `03/Dockerfile` if needed.

**Debugging sequence:**

```bash
# 1. Reproduce flakiness
for i in $(seq 1 30); do
  code=$(curl -so /dev/null -w "%{http_code}" http://localhost:8000/api/users)
  echo "$i $code"
done

# 2. Check all services
docker compose ps
docker compose logs app db

# 3. Network — can app reach db?
docker exec app getent hosts db
docker network inspect $(docker network ls -q | head -1)

# 4. Port mapping — host 8000 → container ???
docker compose config | grep -A2 ports
docker exec app ss -lntp

# 5. Volume — is Postgres data actually persisting?
docker exec db ls -la /var/lib/postgresql/data
```

**Bug checklist (fix ALL — partial fix still fails hidden checks):**
- [ ] `depends_on` without `condition: service_healthy` → race on startup
- [ ] `ports: "8000:80"` but app listens on 8000 inside container
- [ ] App and DB on different networks → intermittent DNS failures
- [ ] Volume mounted to `/var/lib/postgresql` instead of `/var/lib/postgresql/data`
- [ ] Ignore red herring `WARN redis timeout` if Redis is healthy

**Verify before submit:**
```bash
# Must pass 20/20 times
for i in $(seq 1 20); do curl -sf http://localhost:8000/api/users || echo FAIL; done
docker compose restart app
# still works after restart
```

---

### Playbook: Task 04 — Misleading Redis Error (9 pts) ★★★

**Symptom:** Logs scream `could not connect to redis at 127.0.0.1:6379`. Team is debugging Redis. Redis is fine.

**Submit:** `04/compose.yaml` + `04/app.env` or Dockerfile patch.

**Key insight:** The loudest log line is a **red herring**. Prove what is actually broken.

```bash
# Is Redis actually fine?
docker compose ps redis
docker exec redis redis-cli ping          # → PONG

# Can app reach redis by service name?
docker exec app getent hosts redis
docker exec app nc -zv redis 6379

# What about Postgres? (often the real problem)
docker exec app getent hosts db
docker exec app nc -zv db 5432
docker compose logs app | grep -i database

# Is the app even reachable from outside?
docker exec app ss -lntp
curl -v http://localhost:8000/api/users
```

**Bug checklist:**
- [ ] `REDIS_HOST=127.0.0.1` — wrong, but not the only bug
- [ ] App and `db` on different compose networks
- [ ] `DATABASE_URL` uses `localhost` instead of `db`
- [ ] App binds `127.0.0.1:8000` — fix `HOST=0.0.0.0`

**Verify:** `curl /health` AND `curl /api/users` both succeed; `nc -zv db 5432` works from app container.

---

### Playbook: Task 05 — Migration Before Ready (9 pts) ★★★

**Symptom:** Fresh deploy fails. "Just restart app" makes it worse. `migrate` container exited with error.

**Submit:** `05/compose.yaml` + optional `05/fix.sh` (reset corrupt migration state).

```bash
docker compose ps -a                      # check migrate exit code
docker compose logs migrate
docker compose logs db
docker compose logs app

# Is Postgres actually ready when migrate runs?
docker inspect db | jq '.[0].State.Health'
```

**Bug checklist:**
- [ ] `migrate` runs before Postgres accepts connections
- [ ] No `healthcheck` on `db`
- [ ] No `depends_on: db: condition: service_healthy` on `migrate` and `app`
- [ ] Partial migration leaves corrupt schema — may need `docker compose down -v` once, then fix order

**Correct pattern:**
```yaml
db:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U app"]
    interval: 5s
    retries: 5

migrate:
  depends_on:
    db:
      condition: service_healthy

app:
  depends_on:
    migrate:
      condition: service_completed_successfully
    db:
      condition: service_healthy
```

**Verify:** fresh `docker compose down -v && docker compose up --build` → all healthy → `POST /api/users` → `compose down && compose up` → data persists.

---

### Playbook: Task 06 — Slim the Image (6 pts)

**Goal:** Rewrite bloated Dockerfile as multi-stage; final image < 200 MB.

```bash
docker build -t before -f Dockerfile.bloated .
docker images before --format '{{.Size}}'

# After your fix
docker build -t after .
docker images after --format '{{.Size}}'
docker run --rm -p 8000:8000 after
curl http://localhost:8000/
```

**Patterns:**
- Builder stage: install deps, compile if needed
- Runtime stage: `FROM python:3.12-slim` or `FROM gcr.io/distroless/...`
- Copy only the binary/venv artifact, not source tree or build tools

---

### Playbook: Task 07 — Permission Rabbit Hole (9 pts) ★★★

**Symptom:** `Permission denied: '/app/data/state.json'` after adding non-root user.

**Submit:** `07/Dockerfile` + `07/entrypoint.sh` + optional `07/compose.yaml`.

```bash
docker exec app id
docker exec app ls -la /app /app/data
docker inspect app | jq '.[0].Config.User'
docker volume inspect <volname>
```

**Bug chain to unwind:**
1. `COPY . .` leaves files owned by root
2. Named volume mount overwrites `/app/data` with root-owned empty volume
3. Entrypoint tries `chown` but runs as non-root → fails silently
4. App cannot write state file

**Fix pattern in Dockerfile:**
```dockerfile
RUN adduser --disabled-password --gecos "" appuser
COPY --chown=appuser:appuser . .
RUN mkdir -p /app/data && chown appuser:appuser /app/data
USER appuser
```

**Or in entrypoint (must run as root first, then drop):**
```bash
#!/bin/sh
chown -R appuser:appuser /app/data
exec su-exec appuser "$@"
```

**Verify:** `docker exec app id` shows non-root; app creates `/app/data/state.json`.

---

### Playbook: Task 08 — Data Ghost Volume (8 pts) ★★★

**Symptom:** "We added a volume." Data still disappears after redeploy.

**Submit:** `08/compose.yaml`

```bash
# Volume exists on paper?
docker compose config | grep -A5 volumes

# But is it mounted to the RIGHT path inside Postgres?
docker inspect db | jq '.[0].Mounts'

# Postgres official image REQUIRES:
#   /var/lib/postgresql/data
# NOT:
#   /var/lib/postgresql
```

**Test script:**
```bash
docker compose up -d
curl -X POST http://localhost:8000/api/users -d '{"name":"test"}'
docker compose down        # NO -v
docker compose up -d
curl http://localhost:8000/api/users   # must still show "test"
```

**Pass criteria:** named volume declared AND mounted to `/var/lib/postgresql/data`.

---

### Playbook: Task 09 — Exec Rescue, Five Bugs (10 pts) ★★★

**Symptom:** Container running but broken inside. **Cannot rebuild image** — submit `09/fix.sh` only.

**Download fixture and explore:**
```bash
docker run -d --name rescue csot/week2-fixture:broken-v1
docker exec -it rescue sh

# Inside container — find all five bugs:
ls -la /app/data                    # 1. permissions
ls -la /tmp/app.lock                # 2. stale lock
cat /app/.runtime.env               # 3. API_MODE=dev (should be production)
ls -la /app/config.json             # 4. broken symlink
cat /etc/hosts                      # 5. missing internal.api alias
```

**`fix.sh` template (idempotent):**
```bash
#!/bin/sh
set -e
rm -f /tmp/app.lock
chown -R appuser:appuser /app/data
sed -i 's/API_MODE=dev/API_MODE=production/' /app/.runtime.env
ln -sf /app/config.prod.json /app/config.json
echo "127.0.0.1 internal.api" >> /etc/hosts
# restart app process if needed, or touch a trigger file
```

**Verify locally before submit:**
```bash
docker cp fix.sh rescue:/tmp/fix.sh
docker exec rescue sh /tmp/fix.sh
curl http://localhost:<port>/health
```

---

### Playbook: Task 10 — Container Doctor (7 pts)

**Goal:** `10.sh <image>` prints JSON metadata via `docker image inspect`.

```bash
#!/usr/bin/env bash
set -euo pipefail
img="${1:?usage: $0 <image>}"
docker image inspect "$img" | jq '...[fields]...'
```

**Required fields:** `id`, `size` (bytes), `user`, `ports`, `cmd`.

**Edge case:** missing image → exit code 2, message on stderr.

---

### Playbook: Task 11 — SSH: 2 AM Incident (12 pts) ★★★★

**Access:** credentials shown on contest portal after `csot login`.

```bash
ssh incident@week2-debug.csot-devops.devclub.in -p 2222
cd ~/incident-11
```

**On-call sequence:**
```bash
docker compose ps -a
docker compose logs --tail=30 worker app db
curl -s localhost:8000/health              # ok
curl -s localhost:8000/api/users           # 500 — real problem
docker inspect app | jq '.[0].State.Health'
docker network inspect incident-11_default
docker exec app env | grep -E 'DATABASE|REDIS|CACHE'
```

**Bug checklist (6 issues — fix all):**
- [ ] `worker` on wrong network
- [ ] App healthcheck hits port 8080, app listens on 8000
- [ ] `DATABASE_URL` uses `localhost`
- [ ] Invalid `CACHE_TTL=abc` → API 500
- [ ] Worker crash-loop left partial job state — run cleanup
- [ ] Ignore flooded Redis errors in logs (red herring)

**Submit:** copy fixed `11/compose.yaml`, `11/fix.sh`, `11/.env` from VM.

```bash
mkdir -p ~/submissions/11
cp compose.yaml fix.sh .env ~/submissions/11/
csot submit ~/submissions/11 -t 11
```

---

### Playbook: Task 12 — Full Stack Rescue (10 pts) ★★★★

**Goal:** Fix entire `checkout-api` repo. Submit `12/` directory.

**Recommended order of attack:**
```text
1. docker compose config     → parse errors first
2. docker compose build      → Dockerfile issues
3. docker compose up         → runtime issues
4. curl /health              → shallow check
5. curl /api/users           → DB connectivity
6. curl -X POST /api/orders  → worker/async path
7. compose down && up        → persistence
8. docker exec app id        → non-root check
9. docker images             → size check
10. Write NOTES.md           → 5 root-cause/fix pairs
```

**`NOTES.md` template:**
```markdown
## Incident notes — Task 12

### 1. API unreachable from host
- **Symptom:** curl connection refused
- **Command:** `docker exec app ss -lntp`
- **Root cause:** app bound to 127.0.0.1
- **Fix:** HOST=0.0.0.0 in compose environment

### 2. ...
```

**Verify checklist before submit:**
- [ ] `docker compose up --build` → all healthy within 120s
- [ ] `GET /api/users` → 200
- [ ] `POST /api/orders` → 202
- [ ] Data survives `compose down` + `up` (no `-v`)
- [ ] `docker exec app id` → non-root
- [ ] Image < 250 MB
- [ ] `NOTES.md` has ≥5 real root causes (not generic fluff)

---

### Practice fixtures (ship with course repo)

Create `practice/` broken stacks students can run before attempting contest tasks:

| Practice dir | Prepares for | Bugs seeded |
|--------------|--------------|-------------|
| `practice/01-flaky-api/` | Task 03 | 2 of 4 bugs (easier) |
| `practice/02-red-herring-redis/` | Task 04 | misleading log only |
| `practice/03-migration-race/` | Task 05 | migrate before db ready |
| `practice/04-volume-path/` | Task 08 | wrong mount path |
| `practice/05-permissions/` | Task 07 | USER without chown |

```bash
cd practice/01-flaky-api
docker compose up --build
# follow Module 8 playbook for Task 03
```

---

---

## Build 1 — Containerize a Single App

> **Milestone:** mid-week · prepares for contest tasks **01**, **02**.

### Goal

Write a `Dockerfile` and `.dockerignore` for the `app.py` from Module 2. Image builds, runs, responds on port 8000 from the host.

### Step 1: Project layout

```text
build1/
├── app.py
├── Dockerfile
└── .dockerignore
```

### Step 2: Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
ENV HOST=0.0.0.0
ENV PORT=8000
EXPOSE 8000
CMD ["python", "app.py"]
```

### Step 3: `.dockerignore`

```
.env
.git
__pycache__
*.pyc
.venv
```

### Step 4: Build and test

```bash
cd build1
docker build -t csot-build1 .
docker run -d --name build1 -p 8000:8000 csot-build1
curl http://localhost:8000/
docker logs build1
docker stop build1 && docker rm build1
```

### ✅ Build 1 complete when

- `docker build` succeeds
- `curl http://localhost:8000/` returns JSON with `"ok": true`
- `.dockerignore` excludes `.env`
- You can explain why `HOST=0.0.0.0` matters

---

## Build 2 — Multi-Service Compose Stack

> **Milestone:** late-week · prepares for contest tasks **03–05**, **08**.

### Goal

`compose.yaml` with app + Postgres + Redis. App reads/writes DB. Data survives `docker compose down` (without `-v`).

### Step 1: Extend the app (optional DB check endpoint)

Add a `/api/users` that hits Postgres, or use the contest fixture app pattern.

### Step 2: `compose.yaml`

Use the example from [Module 4](#module-4--compose-networking-and-volumes). Key points:

- `DATABASE_URL` uses hostname `db`, not `localhost`
- `HOST=0.0.0.0`
- Named volume at `/var/lib/postgresql/data`
- `healthcheck` on `db` + `depends_on: condition: service_healthy`

### Step 3: Test persistence

```bash
docker compose up -d --build
curl http://localhost:8000/health
# insert or create a user via API
docker compose down
docker compose up -d
# verify data still there
```

### ✅ Build 2 complete when

- All services healthy in `docker compose ps`
- App reaches DB and Redis by service name
- DB data survives `compose down` + `compose up`
- You can draw the network: which services talk to which by name

---

## Weekly Mini-Project — Dockerized App Stack

> **Submission:** GitHub repo link (manually graded, 50 pts) — details in cohort instructions.

### Required contents

| File | Purpose |
|------|---------|
| `Dockerfile` | Multi-stage or slim, non-root |
| `.dockerignore` | No secrets/caches in image |
| `compose.yaml` | App + DB + optional Redis |
| `README.md` | Exact `docker compose up` instructions + screenshots |

### Minimum behavior

- `docker compose up --build` starts everything
- App on documented localhost port
- Config via environment variables
- DB data in named volume
- Healthcheck on app or database
- App runs as non-root
- Logs to stdout/stderr

### Rubric (50 pts)

| Criterion | Points |
|-----------|--------|
| Dockerfile + app starts | 10 |
| `.dockerignore` + reasonable size | 5 |
| Services communicate by name | 10 |
| Persistent DB volume | 5 |
| Env-based config | 5 |
| Healthcheck / readiness | 5 |
| Non-root runtime | 5 |
| README + screenshots | 5 |

---

## Alternative Mini-Project Ideas

1. **Self-hosted Uptime Kuma** — containerized monitoring for 5 URLs
2. **Flask + Postgres URL shortener** — classic CRUD behind Compose
3. **Dev container** — reproducible dev environment in one image
4. **Log aggregator** — app + Redis queue + worker that processes logs
5. **Broken-stack fix log** — document 5 real bugs you fixed in practice fixtures (portfolio piece)

---

## Weekly Quiz Topics

- Image vs container vs registry
- Dockerfile instructions (`FROM`, `COPY`, `RUN`, `CMD`, `USER`)
- Layer caching order
- `.dockerignore` purpose
- `localhost` inside containers vs service-name DNS
- Named volumes vs `docker compose down -v`
- `depends_on` vs `condition: service_healthy`
- Multi-stage builds — why smaller?
- Non-root containers — permission pitfalls
- Trivy basics — what HIGH/CRITICAL means
- Debugging loop — reproduce before fix

---

## Contest reference — tasks, scoring, rules

**12 tasks · 200 points.** Autograded on the server. Partial credit on most tasks. Hidden grader variants on hard tasks (memorized fixes fail).

**Scoring:** sum of per-task bests across attempts, **minus the live-sandbox time penalty** (see below). Tie-break: fewer attempt-penalties → earlier last graded submission.

### The 12 tasks (summary — full specs on contest site)

🔴 = **live VM sandbox** (subject to the time penalty below). The rest are file submissions.

| # | What | Submit as | Pts | Est. time | Solve guide |
|---|------|-----------|-----|-----------|-------------|
| 1 | Hello Container — Dockerfile for provided app | `01/Dockerfile` | 12 | 30 min | Module 8 → Task 01 |
| 2 | Context Leak — `.dockerignore` for secrets/caches | `02/.dockerignore` | 10 | 30 min | Module 8 → Task 02 |
| 3 | 🔴 **The Flapping Stack** — flaky API, 4+ bugs | `03/` | 14 | 60–90 min | Module 8 → Task 03 |
| 4 | 🔴 **Misleading Redis Error** — red herring logs | `04/compose.yaml` + env | 14 | 60 min | Module 8 → Task 04 |
| 5 | 🔴 **Migration Before Ready** — startup order + corrupt state | `05/` | 15 | 75 min | Module 8 → Task 05 |
| 6 | Slim the Image — multi-stage, size cap | `06/Dockerfile` | 12 | 45 min | Module 8 → Task 06 |
| 7 | 🔴 **Permission Rabbit Hole** — USER + volume + entrypoint | `07/` | 16 | 60–90 min | Module 8 → Task 07 |
| 8 | 🔴 **Data Ghost Volume** — volume exists, wrong path | `08/compose.yaml` | 22 | 60 min | Module 8 → Task 08 |
| 9 | 🔴 **Exec Rescue** — fix 5 bugs inside running container | `09/fix.sh` | 14 | 60–90 min | Module 8 → Task 09 |
| 10 | Container Doctor — image inspect JSON script | `10.sh` | 14 | 45 min | Module 8 → Task 10 |
| 11 | 🔴 **SSH: 2 AM Incident** — live VM, 6 bugs | `11/` | 24 | 90–120 min | Module 8 → Task 11 |
| 12 | 🔴 **Full Stack Rescue** — capstone + `NOTES.md` | `12/` | 33 | 90–120 min | Module 8 → Task 12 |

**Total: 200 points.**

### Live-sandbox time penalty (the "2 AM page" clock) 🔴

The eight live VM tasks (03, 04, 05, 07, 08, 09, 11, 12) are graded like a real on-call incident: **the longer your sandbox container stays running before you fix it, the more points that task loses.** This penalty **does reduce your score** — but it can never drop a task below **80%** of its points.

| Container uptime on a task | Points lost |
|---|---|
| under 30 min | 0% |
| 30 min | 2.5% |
| every additional 20 min | +2.5% |
| 170 min and beyond | 20% (capped) |

Key rules:

- The clock only counts **container uptime**. Run `csot challenge stop` when you walk away and the clock **pauses** — thinking/sleeping/idle time is free.
- **Only one sandbox runs at a time.** Starting or switching to another task auto-stops the previous one and pauses its clock.
- File-only tasks (01, 02, 06, 10) have no live sandbox and are **never** time-penalised.
- The penalty is per task: e.g. keeping the Task 11 box (24 pts) up for 90 min costs 10% = ~2 pts; capped loss is 20% = ~5 pts.

Strategy: read the symptoms, plan your fix offline, then start the sandbox and work fast. Stop it the moment you're stuck and want to think.

### Fixture workflow (tasks 03–12)

```bash
# linked from contest task page
curl -fsSL https://csot-devops.devclub.in/week-02/fixtures/03-flapping-stack.tar.gz | tar xz
cd 03-flapping-stack
docker compose up --build
# follow Module 8 playbook for that task number
mkdir -p ~/solutions/03 && cp compose.yaml Dockerfile ~/solutions/03/
csot submit ~/solutions -t 03
```

### Rules at a glance

- **Window**: full Week 2 (exact times on contest site)
- **Rate limit**: 5 submissions/min, 100/contest
- **Submission size**: ≤ 1 MB, ≤ 50 files; per-task timeout up to 120s for capstone
- **Fixtures are meant to be run** — task page shows symptoms, not answers
- **Hidden variants** — grader tests a second breakage; understanding beats memorization
- **Code of honour**: AI and docs fine; copying another student's files is not

### Anti-trivial design (why tasks take real time)

Hard tasks stack multiple bugs, use misleading log lines, require cross-file fixes, and verify stateful behavior (data persists, 20/20 HTTP probes succeed). If you can solve it without running `docker compose up`, the task design failed — tell the organizers.

Tasks **removed** for being too AI-trivial: single env-var paste, "change localhost to db" one-liner, log transcription exercises.

### 60-second start

```bash
curl -fsSL https://csot-devops.devclub.in/install.sh | bash
csot login
csot submit ./my-solutions
csot submit ./my-solutions -t 03
csot history
csot leaderboard --weekly
```

Full specs: **[csot-devops.devclub.in/contest/week-02](https://csot-devops.devclub.in/contest/week-02)**

---

## Resources — What to Read & Study

> **How to use this list:** Don't read everything. Follow the **recommended path** below in order — it maps 1:1 to the modules and contest tasks. Each link has a one-line note on *why* it matters. ⭐ = read this one for sure; the rest are go-deeper/optional.

### Recommended path (in order)

1. **Watch one overview video** (10–15 min) to get the mental model → *Big picture* below.
2. **Read the official "Get started" + Dockerfile basics** → Modules 0–2.
3. **Do a hands-on lab** (Play with Docker / KodeKloud) → cements Modules 1–2.
4. **Read Compose networking + volumes + startup order** → Modules 4–6, the heart of contest tasks 03–08.
5. **Skim image hygiene + security** → Modules 5, 7 (tasks 06, 02).
6. Keep the **official references** open as you debug — don't memorize, look up.

---

### Big picture (start here — pick one)

- ⭐ [Docker in 100 Seconds](https://www.youtube.com/watch?v=Gjnup-PuquQ) (Fireship, ~2 min) — fastest possible mental model.
- ⭐ [Docker Tutorial for Beginners](https://www.youtube.com/watch?v=3c-iBn73dDE) (TechWorld with Nana, ~2 hr) — the single best end-to-end intro; watch the first hour.
- [Containers From Scratch](https://www.youtube.com/watch?v=8fi7uSYlOdc) (Liz Rice) — builds a "container" with raw namespaces/cgroups so you understand what Docker actually does.
- [Containers vs VMs](https://www.redhat.com/en/topics/containers/containers-vs-vms) (Red Hat) — the distinction Module 0 makes, in one page.

### Module 0–1 — Containers & the CLI

- ⭐ [Docker: Get started](https://docs.docker.com/get-started/) — official guided intro; do the workshop.
- ⭐ [docker CLI reference](https://docs.docker.com/reference/cli/docker/) — bookmark; every `run/ps/logs/exec/inspect` flag lives here.
- [What is a container?](https://www.docker.com/resources/what-container/) — concise concept page.
- [How Containers Work](https://wizardzines.com/zines/containers/) (Julia Evans zine) — namespaces, cgroups, and "why is my container weird" explained visually.

### Module 2 — Dockerfiles & build context (tasks 01, 02, 06)

- ⭐ [Dockerfile reference](https://docs.docker.com/reference/dockerfile/) — every instruction (`FROM/COPY/RUN/CMD/ENTRYPOINT/USER/HEALTHCHECK`).
- ⭐ [Building best practices](https://docs.docker.com/build/building/best-practices/) — layer caching order, slim base images, exec-form CMD. Directly relevant to tasks 01 & 06.
- ⭐ [Build context & `.dockerignore`](https://docs.docker.com/build/concepts/context/) — exactly what task 02 (context leak) tests.
- [CMD vs ENTRYPOINT, exec vs shell form](https://docs.docker.com/reference/dockerfile/#cmd) — why shell-form breaks SIGTERM handling.

### Module 3 — Debugging workflow (the core skill for tasks 03–12)

- ⭐ [Explore containers / debug a running container](https://docs.docker.com/engine/containers/run/) — `logs`, `exec`, `inspect` in practice.
- ⭐ [`docker inspect`](https://docs.docker.com/reference/cli/docker/inspect/) + [`jq` tutorial](https://jqlang.org/tutorial/) — read State, Health, Mounts, Networks from JSON.
- [The "bind to 127.0.0.1" trap explained](https://docs.docker.com/engine/network/) — why an app on `127.0.0.1` is unreachable from the host (tasks 04, 11, 12).

### Module 4 — Compose, networking & volumes (tasks 03, 04, 05, 08, 11, 12)

- ⭐ [Compose file reference](https://docs.docker.com/reference/compose-file/) — the spec; services, networks, volumes, env.
- ⭐ [Control startup & shutdown order](https://docs.docker.com/compose/how-tos/startup-order/) — `depends_on` + `condition: service_healthy` / `service_completed_successfully`. **This is task 05.**
- ⭐ [Networking in Compose](https://docs.docker.com/compose/how-tos/networking/) — service-name DNS vs `localhost`, custom networks. **This is tasks 03, 04, 11.**
- ⭐ [Volumes](https://docs.docker.com/engine/storage/volumes/) — named volumes vs bind mounts; `down` vs `down -v`. **This is task 08.**
- ⭐ [Postgres official image](https://hub.docker.com/_/postgres) — read "Where to Store Data": data lives in `/var/lib/postgresql/data` (the exact task-08 trap) and `pg_isready` for healthchecks.

### Module 5 — Image hygiene, multi-stage, non-root (tasks 06, 07)

- ⭐ [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/) — how to ship a tiny runtime image (task 06).
- ⭐ [Run as a non-root user](https://docs.docker.com/build/building/best-practices/#user) — `USER` + `chown` pitfalls. **This is task 07.**
- [Distroless images](https://github.com/GoogleContainerTools/distroless) — the smallest, most secure runtime bases.
- [`docker history` / image size](https://docs.docker.com/reference/cli/docker/image/history/) — see what's bloating your image.

### Module 6 — Healthchecks & 12-factor runtime

- ⭐ [HEALTHCHECK instruction](https://docs.docker.com/reference/dockerfile/#healthcheck) — define readiness; pair with Compose `condition: service_healthy`.
- ⭐ [The Twelve-Factor App](https://12factor.net/) — read **Config**, **Logs**, **Processes**, **Disposability**. Why config goes in env vars and logs go to stdout.
- [Docker and the PID 1 zombie-reaping problem](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/) + [tini](https://github.com/krallin/tini) — why shell-form CMD delays shutdown and how `--init` fixes it.

### Module 7 — Image security (task 06 + good practice)

- ⭐ [Trivy](https://trivy.dev/) — scan images for HIGH/CRITICAL CVEs; see how a slim base cuts findings.
- [Docker engine security](https://docs.docker.com/engine/security/) — capabilities, the docker socket, rootless mode.
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) — the canonical hardening checklist.

### Hands-on labs (do, don't just read)

- ⭐ [Play with Docker](https://labs.play-with-docker.com/) — free in-browser Docker, no install; perfect for Modules 1–4.
- ⭐ [iximiuz Labs](https://labs.iximiuz.com/) — interactive container/networking scenarios that mirror the contest's debugging style.
- [KodeKloud — Docker for the Absolute Beginner](https://kodekloud.com/courses/docker-for-the-absolute-beginner/) — guided course with in-browser labs.
- [docker-curriculum.com](https://docker-curriculum.com/) — free, project-based: single container → multi-service → deploy.

### Books (optional, for depth)

- *Docker Deep Dive* — Nigel Poulton (the standard reference; very readable).
- *The Docker Book* — James Turnbull (build-up from basics).

### Keep open while doing the contest (quick reference)

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/) · [Compose file reference](https://docs.docker.com/reference/compose-file/) · [docker CLI](https://docs.docker.com/reference/cli/docker/) · [compose CLI](https://docs.docker.com/reference/cli/docker/compose/)
- This guide's **Module 3** (debugging loop + toolkit) and **Module 8** (per-task playbooks).

---

**Next week: Week 3 — CI/CD, GitHub Actions & GHCR.** You will take the Docker image from this week and automate build + push on every merge to `main`.
