# CSOT 2026 — DevOps Vertical

**CAIC Summer of Tech | 6-Week Project-Based DevOps Program**

> Build, ship, and run **7 portfolio projects** in 6 weeks — from Linux scripts to a fully observable cloud-native app.

---

## 📖 Documentation

- **[Comprehensive Guide](./Comprehensive-Guide.md)** — full curriculum design, rubrics, mentor notes
- **[Weekly content (`content/`)](./content/)** — one markdown guide per week (modules, builds, mini-projects, contest pitch)

### Weekly content (start here each week)


| Week | Guide                                                                                                        | Focus                                                                                                                      |
| ---- | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| 1    | [week-01-linux-networking-git.md](./content/week-01-linux-networking-git.md)                                 | Linux, shell, networking, Git, nginx, systemd · [Week 1 contest](https://csot-devops.devclub.in/contest/week-01) (100 pts) |
| 2    | [week-02-cicd-quality-registries.md](./content/week-02-cicd-quality-registries.md)                           | CI/CD, GitHub Actions, quality gates, GHCR                                                                                 |
| 3    | [week-03-containers-compose-security.md](./content/week-03-containers-compose-security.md)                   | Docker, Compose, image security                                                                                            |
| 4    | [week-04-kubernetes-helm-gitops.md](./content/week-04-kubernetes-helm-gitops.md)                             | Kubernetes, Helm, GitOps                                                                                                   |
| 5    | [week-05-iac-aws-cloudflare-finops.md](./content/week-05-iac-aws-cloudflare-finops.md)                       | Terraform, AWS, Cloudflare, FinOps                                                                                         |
| 6    | [week-06-observability-sre-devsecops-capstone.md](./content/week-06-observability-sre-devsecops-capstone.md) | Observability, SRE, capstone                                                                                               |


Read the week's guide for teaching material. 

**Week 1 includes a mandatory contest** (100 points, runs the full week) at **[csot-devops.devclub.in](https://csot-devops.devclub.in)** 

---

## 🗓 The 6 Weeks

### Week 1 — Linux, Networking, Git & Sysadmin Foundations

 **[Read Week 1 content](./content/week-01-linux-networking-git.md)** · 🏆 **[Week 1 contest](https://csot-devops.devclub.in/contest/week-01)** (12 tasks, **100 points**, autograded)

Master the OS and the network — the bedrock everything else stacks on. Topics covered:

- **DevOps & System Design Primer** — CALMS framework, DevOps lifecycle, monolith vs microservices, CAP theorem, scalability, load balancing (L4 vs L7), reverse proxy vs API gateway
- **Linux Essentials** — filesystem hierarchy, permissions (`chmod`, `chown`), processes (`ps`, `top`, `htop`), package managers (`apt` / `dnf` / `brew`), text manipulation (`grep`, `awk`, `sed`, `cut`, `sort`, `uniq`), environment & `PATH`
- **Shell Scripting & Automation** — bash basics, defensive scripting (`set -euo pipefail`), `getopts`, `trap`, cron jobs, 12-Factor *Config*
- **Long-Lived Services** — `tmux` (windows, panes, detach/attach), background jobs (`&`, `nohup`, `disown`), `systemd` services + timers, `journalctl`
- **Networking Fundamentals** — TCP vs UDP, ports, SSH + key auth, diagnostic tools (`ping`, `curl`, `dig`, `nslookup`, `ss`, `traceroute`), DNS records (A, CNAME, MX, TXT, NS), firewalls, HTTP basics
- **Web Servers, Proxies & TLS** — forward vs reverse proxy, nginx (server / location blocks, `proxy_pass`, upstream / load balancing), Apache comparison, Caddy mention, Let's Encrypt + Certbot
- **Git & GitHub Mastery** — branching, merge vs rebase, conflict resolution, `git stash` / `reflog` / `bisect`, GitHub Flow vs Git Flow vs Trunk-Based, conventional commits
- **Secrets Hygiene** — `.env` pattern, pre-commit hooks, **TruffleHog** secret scanning, leak response (rotate, revoke)

### Week 2 — CI/CD, Quality Engineering & Registries

Automate everything from commit to artifact, and never let a broken or insecure build ship. Topics covered:

- **Why CI/CD** — pipeline stages, Continuous Integration vs Delivery vs Deployment, tool landscape (Actions, Jenkins, GitLab CI, CircleCI)
- **GitHub Actions Anatomy** — workflows, jobs, steps, runners, triggers (`push`, `pull_request`, `schedule`, `workflow_dispatch`), the Action marketplace + SHA pinning
- **Workflow Patterns** — matrix builds, dependency caching, secrets + environments, composite actions, reusable workflows (`workflow_call`), `needs:`, conditionals
- **Quality Gates** — linting (ESLint / Flake8 / golangci-lint), unit + coverage tests, **JSON-schema contract testing**, **TruffleHog** secret scan, dependency auditing (`npm audit`, `pip-audit`, `govulncheck`), Dependabot, branch protection
- **Continuous Delivery: GHCR** — registry comparison, GitHub Container Registry setup, `GITHUB_TOKEN`, `docker/build-push-action`, image tagging (`latest` + commit SHA), making images public
- **Repo Hygiene & Release Automation** — semantic versioning, conventional commits, `semantic-release`, `release-please`, auto-CHANGELOG, required status checks
- **Jenkins Awareness** — architecture (master/agent), Jenkinsfile (declarative pipeline), when companies still use Jenkins
- **Other CI Tools (Brief)** — GitLab CI, CircleCI, Travis, Drone CI

### Week 3 — Containers, Compose & Image Security

Package once, run anywhere — securely and slim. Topics covered:

- **Container Fundamentals** — Linux namespaces, cgroups, containers vs VMs, Docker architecture (daemon, containerd, runc), images vs containers, OCI standard
- **Writing Dockerfiles** — layer caching + instruction ordering, **multi-stage builds**, `.dockerignore`, base image choices (slim / alpine / distroless), `HEALTHCHECK`, image-digest pinning
- **Container Runtime** — Docker networking (bridge, host, custom), volumes (bind, named, tmpfs), environment vars, debugging (`logs`, `exec`, `inspect`, `dive`)
- **Multi-Service with `docker compose`** — services, networks, volumes, `depends_on` + healthcheck conditions, override files, dev vs prod compose
- **Image Security & Supply Chain** — **Trivy** vulnerability scanning, **Syft** for SBOM, **cosign** image signing, multi-arch builds with `buildx`
- **Registries** — GHCR (primary), Docker Hub, ECR, Quay, Harbor
- **12-Factor App Recap (Container Edition)** — config via env, log to stdout, stateless processes, dev/prod parity
- **Cloudflare Tunnels** — public HTTPS for local containers with `cloudflared`, quick vs named tunnels

### Week 4 — Kubernetes, Helm & GitOps

Run containers at scale, reliably, declaratively. Topics covered:

- **Why Orchestration** — limitations of compose, Kubernetes architecture (control plane: API server, etcd, scheduler, controller manager; nodes: kubelet, kube-proxy), the reconciliation loop
- **Local Cluster Setup** — Kind vs Minikube vs k3s, multi-node `kind-config.yaml`, nginx-ingress controller install
- **Core Workload Objects** — Pod, ReplicaSet, Deployment, StatefulSet, DaemonSet, Job, CronJob, Namespace
- **kubectl Fluency** — `get` / `describe` / `apply` / `logs` / `exec` / `port-forward`, `kubectl explain`, useful aliases + completion
- **Configuration & State** — ConfigMaps, Secrets (and why they aren't encryption), resource requests vs limits, PersistentVolumes + PVCs, when to use StatefulSets
- **Networking & Reliability** — Service types (ClusterIP / NodePort / LoadBalancer / ExternalName), Ingress (nginx-ingress), liveness / readiness / startup probes, PodDisruptionBudgets, Network Policies (brief)
- **Helm** — charts, releases, templating, `values.yaml`, sub-charts, `helm install/upgrade/rollback/template/lint`
- **GitOps with ArgoCD** — philosophy, `Application` CRD, sync policies (auto-sync, prune, self-heal), drift detection, sync waves + hooks, Flux comparison
- **Debugging Kubernetes** — the big four (CrashLoopBackOff, ImagePullBackOff, OOMKilled, Pending), reading events, `kubectl debug` for distroless

### Week 5 — IaC, Cloudflare & FinOps

Real infrastructure, declaratively managed, cost-aware. Topics covered:

- **Cloud Safety Briefing** — billing alerts, MFA, IAM users (never root), `terraform destroy` discipline, free-tier limits + surprise-cost services (NAT, EKS control plane)
- **IaC Principles** — declarative vs imperative, idempotency, drift detection, tool comparison (Terraform, Pulumi, CloudFormation, CDK)
- **Terraform Basics** — providers, resources, variables, outputs, locals, data sources, the workflow (`init` / `plan` / `apply` / `destroy`), state file
- **Terraform at Scale** — modules (writing + consuming), workspaces, `for_each` / `count` / dynamic blocks, **remote state** (Terraform Cloud free tier OR S3 + DynamoDB lock)
- **AWS Fundamentals** — IAM (least privilege), VPC (subnets, route tables, IGW, NAT), EC2, S3, RDS, Security Groups, EKS overview
- **GCP Awareness (Short)** — GKE / Cloud Run / Cloud Storage equivalents, when to pick GCP
- **Cloudflare Deep Dive** — DNS records, proxy modes, Tunnels (`cloudflared`), Access (Zero Trust), Pages, Email Routing, R2 (S3-compatible), Workers
- **Multi-Provider Terraform** — combining `aws` + `cloudflare` providers in one repo, cross-provider dependencies
- **FinOps Basics** — Cost Explorer, AWS Budgets with alerts, Spot vs On-Demand vs Reserved, right-sizing, cost allocation tags, Lambda vs Fargate vs EC2 cost lens, `infracost`
- **Ansible Awareness** — config management vs provisioning, playbook anatomy, when to use alongside Terraform

### Week 6 — Observability, SRE, DevSecOps & Capstone

Make it visible, keep it alive, ship it safely — then deliver the final project. Topics covered:

- **The 3 Pillars of Observability** — metrics vs logs vs traces, monitoring vs observability
- **Prometheus** — pull architecture, exporters, Alertmanager, metric types (Counter / Gauge / Histogram / Summary), instrumenting Python/Node/Go apps with `/metrics`
- **PromQL** — instant vectors, range vectors, `rate()`, `sum by()`, `histogram_quantile()`, common recipes
- **Grafana** — data sources, dashboards, panels, variables, importing community dashboards, dashboard-as-code (JSON export), alerting
- **Putting It on Kubernetes** — `kube-prometheus-stack` Helm chart, `ServiceMonitor` CRD, **RED method**, **USE method**, **4 Golden Signals**
- **Logging & Tracing** — **Loki + Promtail** (LogQL), ELK mention, **OpenTelemetry** standard, **Jaeger** trace UI, structured (JSON) logging
- **SRE Deep Dive** — **SLI / SLO / SLA**, error budgets + burn rate math, toil reduction, multi-window multi-burn-rate alerts, free Google SRE book chapters
- **Incident Response** — severity levels (Sev1–Sev4), on-call rotation basics, **runbook writing**, postmortem culture (blameless), action items
- **DevSecOps** — **SAST** (Semgrep, Bandit, gosec), **DAST** (OWASP ZAP), dependency scanning (Snyk, Dependabot), container scanning (Trivy recap), supply chain (SBOM with Syft, signing with cosign), OWASP Top 10
- **Chaos Engineering (Intro)** — the Netflix story, Litmus / Chaos Mesh, a first chaos experiment
- **Capstone Project** — final week dedicated to polishing and submitting the natural conclusion of everything you've built since Week 1

---

## 🧰 Topics Covered

- **Linux** — shell, processes, permissions, networking, **nginx**, **systemd**, **tmux**
- **Git** — branching, merging, rebasing, **GitHub Flow**, secrets hygiene
- **CI/CD** — **GitHub Actions**, Jenkins (awareness), reusable workflows, quality gates
- **Quality** — lint, test, JSON-schema contracts, **TruffleHog**, dependency audits
- **Containers** — **Docker**, multi-stage builds, compose, **Trivy**, **Syft (SBOM)**
- **Orchestration** — **Kubernetes** (Kind), **Helm**, **ArgoCD GitOps**
- **IaC** — **Terraform** (Docker + Cloudflare + AWS providers), Ansible (awareness)
- **Cloud** — AWS (awareness/bonus), **Cloudflare** (DNS, Tunnels, Access, Pages)
- **Observability** — **Prometheus**, **Grafana**, **Loki**, OpenTelemetry, Jaeger
- **SRE** — SLI/SLO/SLA, error budgets, runbooks, postmortems, on-call
- **DevSecOps** — SAST, DAST, dependency scanning, image signing, supply chain
- **FinOps** — Cost Explorer, Budgets, Spot, right-sizing
- **System Design** — load balancing, scalability, proxies, microservices

---

## 💡 Project-First Philosophy

- Every week is built around **one buildable mini-project** + several alternative project ideas you can pick from
- Each project has a **🟢 Local Track** and a **🟡 Cloud Track (optional bonus)**
- Capstone in Week 6 ties everything together into one polished, deployable system
- **7 portfolio pieces** by the end — each with a public URL or demoable GitHub repo

---

## 💰 Cost

**Zero credit card required** for the local track of every week. The full program runs on:

- Your laptop (Linux / WSL / Mac)
- Free GitHub account (with free Actions runners + GHCR)
- Free Cloudflare account (DNS + Tunnels + Pages + Access)
- Free Terraform Cloud account (remote state)

Cloud bonuses (AWS / GCP / managed Kubernetes) are **optional** for students who have credits (AWS Educate, GitHub Student Pack, Oracle Free Tier, etc.).

---

## ⏱ Time Commitment

- ~7–9 hours/week, self-paced
- Async via the content repo; live sessions only for the kickoff, mid-program AMA, capstone kickoff, and demo day

---

## 🚀 Getting Started

1. Read the **[Comprehensive Guide](./Comprehensive-Guide.md)** for the full picture
2. Open **[Week 1](./content/week-01-linux-networking-git.md)** — modules, builds, and mini-project live in `**content/`** (one file per week)
3. Set up Linux / WSL, install Git, Docker, GitHub CLI (see Week 1's *Tools & Software Used*)
4. Complete the **mandatory** **[Week 1 contest](https://csot-devops.devclub.in/contest/week-01)** (`csot` CLI, 100 points, open all week) and ship the mini-project
5. Each following week: read the matching `content/week-0N-*.md` guide before that week's builds

---

*Organized by CSOT (CAIC Summer of Tech) — DevOps Vertical, 2026 cohort.*