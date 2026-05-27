# Week 1 — Linux, Networking, Git & Sysadmin Foundations

> **Theme**: Master the OS and the network. Every DevOps skill stacks on this foundation.
>
> **Time Budget**: ~8 hours (reading 3h + builds 3h + mini-project 2h)

This file is **Week 1 of the CSOT DevOps weekly curriculum**. The full six-week syllabus lives in the repo root **[README.md](../README.md)** under *The 6 Weeks* — each week has its own guide under `**[content/](../content/)`** (this file, then Week 2–6).

---

## Table of Contents

1. [Tools & Software Used](#tools--software-used)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Learning Outcomes](#learning-outcomes)
4. **[Week 1 Contest — mandatory (100 pts)](#week-1-contest--mandatory-100-points)**
5. [Module 0 — DevOps & System Design Primer](#module-0--devops--system-design-primer)
6. [Module 1 — Linux Essentials](#module-1--linux-essentials)
7. [Module 2 — Shell Scripting & Automation](#module-2--shell-scripting--automation)
8. [Module 3 — Long-Lived Services (tmux & systemd)](#module-3--long-lived-services-tmux--systemd)
9. [Module 4 — Networking Fundamentals](#module-4--networking-fundamentals)
10. [Module 5 — Web Servers, Reverse Proxies & TLS](#module-5--web-servers-reverse-proxies--tls)
11. [Module 6 — Git & GitHub Mastery](#module-6--git--github-mastery)
12. [Module 7 — Secrets Hygiene](#module-7--secrets-hygiene)
13. [Build 1 — System Health Monitor (Wed)](#build-1--system-health-monitor-wed)
14. [Build 2 — App Behind nginx with HTTPS (Fri)](#build-2--app-behind-nginx-with-https-fri)
15. [Weekly Mini-Project — DevOps Toolkit Repo](#weekly-mini-project--devops-toolkit-repo)
16. [Alternative Mini-Project Ideas](#alternative-mini-project-ideas)
17. [Weekly Quiz Topics](#weekly-quiz-topics)
18. [Contest reference — tasks, scoring, rules](#contest-reference--tasks-scoring-rules)
19. [Resources](#resources)

---

## Tools & Software Used

This week's toolbox.


| Tool                | Purpose                        | Install (Linux/WSL)                              | Install (Mac)             | Install (Windows)                               |
| ------------------- | ------------------------------ | ------------------------------------------------ | ------------------------- | ----------------------------------------------- |
| **Linux / WSL2**    | The OS we live in              | Already on Ubuntu/Kali/Debian                    | macOS is Unix-like; skip  | `wsl --install -d Ubuntu` (PowerShell as admin) |
| **bash**            | Default shell                  | Built-in                                         | Built-in                  | Inside WSL                                      |
| **git**             | Version control                | `sudo apt install git`                           | `brew install git`        | Git for Windows + Git Bash, or use WSL          |
| **gh** (GitHub CLI) | Auth + PRs from terminal       | `sudo apt install gh`                            | `brew install gh`         | `winget install GitHub.cli`                     |
| **tmux**            | Persistent terminal sessions   | `sudo apt install tmux`                          | `brew install tmux`       | Inside WSL                                      |
| **systemd**         | Service manager                | Built-in on modern distros                       | N/A (use launchd)         | Inside WSL                                      |
| **nginx**           | Web server / reverse proxy     | `sudo apt install nginx`                         | `brew install nginx`      | Inside WSL                                      |
| **certbot**         | Let's Encrypt TLS certificates | `sudo apt install certbot python3-certbot-nginx` | `brew install certbot`    | Inside WSL                                      |
| **curl, wget**      | HTTP clients                   | Pre-installed                                    | Pre-installed             | Inside WSL                                      |
| **dig, nslookup**   | DNS lookup                     | `sudo apt install dnsutils`                      | Pre-installed (`bind`)    | Inside WSL                                      |
| **ss, netstat**     | Socket inspection              | Pre-installed                                    | `netstat` only            | Inside WSL                                      |
| **htop**            | Better `top`                   | `sudo apt install htop`                          | `brew install htop`       | Inside WSL                                      |
| **jq**              | JSON processor                 | `sudo apt install jq`                            | `brew install jq`         | Inside WSL                                      |
| **trufflehog**      | Secret scanner                 | See [Module 7](#module-7--secrets-hygiene)       | `brew install trufflehog` | `pip install truffleHog`                        |


**Verification command** (run after installing everything):

```bash
for tool in git gh tmux nginx certbot curl dig ss htop jq trufflehog; do
  command -v "$tool" >/dev/null 2>&1 && echo "✓ $tool" || echo "✗ $tool MISSING"
done
```

Expected output:

```
✓ git
✓ gh
✓ tmux
✓ nginx
✓ certbot
✓ curl
✓ dig
✓ ss
✓ htop
✓ jq
✓ trufflehog
```

---

## Prerequisites & Setup

- A Linux box: (Any one)
  - 1. If on Windows : Do install WSL2  [https://youtu.be/J8cy6MDkacI?si=XP1w5gTWI820eoQ_](https://youtu.be/J8cy6MDkacI?si=XP1w5gTWI820eoQ_)
  - 1. If you have linux , you are good to go.
  - 1. Free Tier cloud compute like Azure (100$ credits)
- 4 GB RAM minimum
- A GitHub account (free)
- Cloudflare Account
- (Optional) A domain name for Build 2's HTTPS — `.tech`  Free with github education on [https://get.tech/github-student-developer-pack](https://get.tech/github-student-developer-pack)

All code snippets are Linux bash code you need either WSL2 or Linux Operating System

---

## Learning Outcomes

By the end of this week, you can:

- Navigate Linux confidently; read and set file permissions; manage processes
- Write idiomatic shell scripts with proper error handling
- Use `systemd` to run long-lived services; use `tmux` for persistent SSH sessions
- Understand TCP/UDP, DNS, HTTP; debug network issues with `dig`, `ss`, `curl`
- Configure **nginx as a reverse proxy with HTTPS** in front of a backend app
- Use Git for branching, merging, rebasing, and conflict resolution
- Manage secrets safely — never commit them; scan history with **TruffleHog**

---

## Week 1 Contest — mandatory (100 points)

> **This contest is mandatory for every Week 1 participant.** It runs for the **whole week** . It is worth **100 points** on the cohort leaderboard and is graded automatically on **[csot-devops.devclub.in](https://csot-devops.devclub.in)**.


|              |                                                                                                            |
| ------------ | ---------------------------------------------------------------------------------------------------------- |
| **Platform** | [csot-devops.devclub.in](https://csot-devops.devclub.in) — sign in with DevClub                            |
| **Tasks**    | 12 shell/sysadmin challenges (see [task index](https://csot-devops.devclub.in/contest/week-01))            |
| **Points**   | **100 total** (7–10 pts per task; partial credit on many tasks)                                            |
| **When**     | Open **all week** — check the site for the exact open/close times for your cohort                          |
| **How**      | `csot` CLI — install once, `csot login`, then `csot submit ./solutions` (whole folder or `-t NN` per task) |


Plan time alongside the modules below — several contest tasks map directly to Module 2 (shell), Module 3 (systemd), and Module 5 (nginx/TLS).

```bash
curl -fsSL https://csot-devops.devclub.in/install.sh | bash
csot login
csot submit ./my-solutions          # all tasks when ready
csot submit ./my-solutions -t 02  # or one task at a time
```

Full task list, skeleton examples, visible/hidden grader rules, and tie-break details: **[Contest reference](#contest-reference--tasks-scoring-rules)** at the end of this doc and on the **[contest site](https://csot-devops.devclub.in/contest/week-01)** (source of truth).

---

## Module 0 — DevOps & System Design Primer

### What is DevOps?

DevOps is the cultural and technical practice of bringing **Dev** (people who build) and **Ops** (people who keep things running) together. The goal: ship software faster *and* more reliably.

The acronym **CALMS** captures the pillars:

- **C**ulture: collaboration over silos
- **A**utomation: humans don't do repetitive tasks; machines do
- **L**ean: small batches, fast feedback
- **M**easurement: you can't improve what you don't measure
- **S**haring: knowledge, tools, blame-free postmortems

### The DevOps Lifecycle

```
┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
│ PLAN │ → │ CODE │ → │ BUILD│ → │ TEST │ → │SHIP/ │ → │MONITOR│
│      │   │      │   │      │   │      │   │DEPLOY│   │       │
└──────┘   └──────┘   └──────┘   └──────┘   └──────┘   └──┬───┘
   ▲                                                      │
   └──────────────────────────────────────────────────────┘
                  feedback loop
```

Every box in this picture is a problem we'll solve this program:

- Week 1: the OS those services run on
- Week 2: CODE → BUILD → TEST automation (CI/CD)
- Week 3: BUILD (containers)
- Week 4: SHIP (Kubernetes)
- Week 5: SHIP infrastructure too (Terraform)
- Week 6: MONITOR (Prometheus + Grafana)

### System Design Basics

Before we automate, you need to know what you're automating.

**Monolith vs Microservices:**

- **Monolith**: one big codebase, deployed as one unit. Simple to start. Hard to scale individual parts.
- **Microservices**: many small services that talk over the network. Scales well. Operationally complex (this is why DevOps exists).

**Scalability:**

- **Vertical** (scale up): bigger server. Hits a wall fast.
- **Horizontal** (scale out): more servers. Requires stateless apps + a load balancer.

**Load Balancing — L4 vs L7:**

- **L4** (TCP/UDP): fast, dumb. Knows IPs and ports. Used in front of databases, raw TCP.
- **L7** (HTTP): smart. Knows URLs, headers, cookies. Used in front of web apps. nginx (Module 5) is L7.

**Reverse Proxy:**
A server that sits in front of your real backend and forwards requests to it. The client never talks to the backend directly. This gives you:

- TLS termination (HTTPS at the edge)
- Load balancing
- Caching
- Hiding backend topology

**API Gateway:**
A reverse proxy + extras (auth, rate limiting, request transformation). Same idea, fancier.

**Service Mesh:**
Reverse proxies *inside* the cluster, between every service. Istio / Linkerd. Don't worry about this for now.

### Going Deeper

- *The Phoenix Project* by Gene Kim — the novel that explains DevOps culture
- *Designing Data-Intensive Applications* by Martin Kleppmann — Ch 1 for system design fundamentals
- [12-Factor App](https://12factor.net/) — the canonical "how to build cloud-friendly apps" doc

---

## Module 1 — Linux Essentials

> If you don't know Linux, you don't know DevOps. Period.

### Filesystem Hierarchy

```
/                # root of everything
├── bin/         # essential user binaries (ls, cp, mv)
├── boot/        # kernel + bootloader
├── dev/         # device files (/dev/sda, /dev/null)
├── etc/         # system-wide config (nginx, systemd, ssh)
├── home/        # user home directories (/home/sumit)
├── lib/         # shared libraries
├── opt/         # optional / third-party software
├── proc/        # virtual filesystem (process info)
├── root/        # root user's home
├── tmp/         # temporary files (wiped on reboot)
├── usr/         # user programs (/usr/bin, /usr/local)
└── var/         # variable data (logs, mail, cache)
```

**Key paths to remember:**

- `/etc/nginx/` — nginx config
- `/etc/systemd/system/` — your custom systemd units
- `/var/log/` — system and app logs
- `/proc/cpuinfo`, `/proc/meminfo` — live system info

### Permissions

Every file has 3 permission triplets: **owner**, **group**, **others**. Each triplet has **r** (read), **w** (write), **x** (execute).

```bash
ls -l /etc/passwd
# -rw-r--r-- 1 root root 2849 May 25 08:00 /etc/passwd
#  └┬┘└┬┘└┬┘
#   │  │  └── others: read only
#   │  └───── group:  read only
#   └──────── owner:  read + write
```

Numeric form (each triplet is a number 0-7):

- `r` = 4, `w` = 2, `x` = 1
- `rwx` = 7, `rw-` = 6, `r--` = 4

```bash
chmod 755 myscript.sh          # rwxr-xr-x
chmod +x myscript.sh           # add execute for all
chmod u+x,go-w myscript.sh     # owner +x, group/others -w
chown alice:dev myscript.sh    # change owner + group
```

### Processes

```bash
ps aux                # all processes, BSD-style
ps -ef                # all processes, System-V style
ps -ef | grep nginx   # find nginx processes
top                   # live process view (q to quit)
htop                  # prettier top
kill 1234             # send SIGTERM (polite ask)
kill -9 1234          # send SIGKILL (force, no cleanup)
killall nginx         # by name
```

**Try it yourself:** Find the PID of your shell:

```bash
echo $$
# 9234
ps -p $$
#   PID TTY          TIME CMD
#  9234 pts/0    00:00:00 bash
```

### Package Managers


| Distro                 | Manager  | Install                | Search            | Update                                |
| ---------------------- | -------- | ---------------------- | ----------------- | ------------------------------------- |
| Ubuntu / Debian / Kali | `apt`    | `sudo apt install pkg` | `apt search pkg`  | `sudo apt update && sudo apt upgrade` |
| Fedora / RHEL          | `dnf`    | `sudo dnf install pkg` | `dnf search pkg`  | `sudo dnf upgrade`                    |
| Arch                   | `pacman` | `sudo pacman -S pkg`   | `pacman -Ss pkg`  | `sudo pacman -Syu`                    |
| macOS                  | `brew`   | `brew install pkg`     | `brew search pkg` | `brew upgrade`                        |


### Text Manipulation: The Magic Five

These are the bread-and-butter of every shell script:

`**grep**` — search for patterns:

```bash
grep "ERROR" /var/log/syslog
grep -i "error" /var/log/syslog        # case insensitive
grep -r "TODO" .                       # recursive
grep -v "DEBUG" app.log                # invert (lines NOT matching)
grep -c "200" access.log               # count matches
```

`**awk**` — column-based processing:

```bash
# Print the 1st column of /etc/passwd (separated by ':')
awk -F: '{print $1}' /etc/passwd
# root
# daemon
# sumit
# ...

# Sum a column of numbers
echo -e "10\n20\n30" | awk '{sum+=$1} END {print sum}'
# 60
```

`**sed**` — stream editor:

```bash
sed 's/foo/bar/g' file.txt             # replace all foo with bar
sed -i 's/foo/bar/g' file.txt          # edit in place
sed -n '5,10p' file.txt                # print lines 5-10
```

`**cut**` — extract columns:

```bash
cut -d: -f1 /etc/passwd                # 1st field, : delimiter
echo "hello world" | cut -c1-5         # first 5 characters
```

`**sort | uniq -c | sort -rn**` — the classic top-N pattern:

```bash
# Top 5 most-frequent IPs in an nginx access log
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -5
# 1547 192.168.1.100
#  892 10.0.0.5
#  611 203.0.113.42
#  ...
```

### Environment & PATH

```bash
echo $HOME              # /home/sumit
echo $PATH              # colon-separated list of binary search paths
export FOO=bar          # set env var for this shell
echo $FOO               # bar
env                     # show all env vars
```

To make an env var persistent, add to `~/.bashrc` or `~/.zshrc`:

```bash
echo 'export EDITOR=vim' >> ~/.bashrc
source ~/.bashrc        # reload
```

---

## Module 2 — Shell Scripting & Automation

### Anatomy of a Bash Script

```bash
#!/usr/bin/env bash
# ^^^^^^^^^^^^^^^^^ shebang — tells the kernel which interpreter to use

set -euo pipefail
# -e: exit on any error
# -u: error on unset variable
# -o pipefail: fail if any command in a pipe fails

# Variables
NAME="World"
COUNT=5

# Function
greet() {
    local who="$1"           # $1 is the first argument
    echo "Hello, $who!"
}

# Loop
for i in $(seq 1 $COUNT); do
    greet "$NAME #$i"
done
```

Save as `hello.sh`, then:

```bash
chmod +x hello.sh
./hello.sh
```

Output:

```
Hello, World #1!
Hello, World #2!
Hello, World #3!
Hello, World #4!
Hello, World #5!
```

### Conditionals

```bash
if [[ -f /etc/passwd ]]; then
    echo "passwd exists"
fi

# Test operators:
# -f file exists
# -d directory exists
# -z string is empty
# -n string is non-empty
# == string equal, != not equal
# -eq numeric equal, -ne, -lt, -gt, -le, -ge

case "$1" in
    start) echo "starting" ;;
    stop)  echo "stopping" ;;
    *)     echo "usage: $0 start|stop"; exit 1 ;;
esac
```

### Loops

```bash
# C-style
for ((i=0; i<5; i++)); do
    echo "iteration $i"
done

# Over a list
for file in *.log; do
    echo "found: $file"
done

# Reading a file line by line
while IFS= read -r line; do
    echo "line: $line"
done < input.txt
```

### Argument Parsing with `getopts`

```bash
#!/usr/bin/env bash
set -euo pipefail

usage() { echo "Usage: $0 [-v] [-n NAME] [-c COUNT]"; exit 1; }

VERBOSE=0
NAME="World"
COUNT=1

while getopts "vn:c:h" opt; do
    case $opt in
        v) VERBOSE=1 ;;
        n) NAME="$OPTARG" ;;
        c) COUNT="$OPTARG" ;;
        h|*) usage ;;
    esac
done

[[ $VERBOSE -eq 1 ]] && echo "DEBUG: name=$NAME count=$COUNT"

for i in $(seq 1 $COUNT); do
    echo "Hello, $NAME #$i"
done
```

Run:

```bash
./greet.sh -v -n Sumit -c 3
# DEBUG: name=Sumit count=3
# Hello, Sumit #1
# Hello, Sumit #2
# Hello, Sumit #3
```

### Error Handling with `trap`

```bash
#!/usr/bin/env bash
set -euo pipefail

cleanup() {
    echo "cleaning up tmp files..."
    rm -f /tmp/myapp.lock
}
trap cleanup EXIT     # always run cleanup when script exits

# do work
touch /tmp/myapp.lock
echo "running..."
sleep 2
# cleanup runs automatically
```

### Cron Jobs

Schedule things to run periodically:

```bash
crontab -e         # edit your cron table

# Syntax: minute hour day_of_month month day_of_week command
# Every 5 minutes:
*/5 * * * * /home/sumit/scripts/health.sh >> /var/log/health.log 2>&1

# Daily at 3 AM:
0 3 * * * /home/sumit/scripts/backup.sh

# Every Monday at 9 AM:
0 9 * * 1 /home/sumit/scripts/weekly-report.sh
```

For better scheduling (and logging), use `**systemd` timers** (next module).

### The 12-Factor App: Config

> Store config in the environment, not in code.

**Bad** (hardcoded):

```python
db_url = "postgres://user:secretpassword@localhost/mydb"
```

**Good** (from environment):

```python
import os
db_url = os.environ["DATABASE_URL"]
```

In Bash:

```bash
: "${DATABASE_URL:?DATABASE_URL must be set}"
echo "Connecting to $DATABASE_URL"
```

If `DATABASE_URL` is unset, this dies immediately with a helpful error.

---

## Module 3 — Long-Lived Services (tmux & systemd)

### The Problem

You SSH into a server, start a process, close your laptop → process dies. Bad.

Two solutions:

1. `**tmux**` — for interactive sessions you want to resume
2. `**systemd**` — for production services that should restart on crash and boot

### tmux: Persistent Terminal Sessions

```bash
tmux new -s work       # create session named "work"
# (do stuff)
# detach with Ctrl+b, then d

tmux ls                # list sessions
# work: 1 windows (created Mon May 25 08:30:00 2026)

tmux attach -t work    # reattach
tmux kill-session -t work
```

**Essential keybindings** (prefix is `Ctrl+b`):


| Keybinding       | Action                 |
| ---------------- | ---------------------- |
| `Ctrl+b c`       | New window             |
| `Ctrl+b n` / `p` | Next / previous window |
| `Ctrl+b "`       | Split horizontally     |
| `Ctrl+b %`       | Split vertically       |
| `Ctrl+b arrow`   | Move between panes     |
| `Ctrl+b d`       | Detach                 |
| `Ctrl+b ?`       | Show all keybindings   |


**Try it yourself:** SSH into a remote machine, run `tmux new -s test`, start `top`, detach with `Ctrl+b d`, log out, log back in, `tmux attach -t test`. Your `top` is still running.

### Background Jobs (Quick & Dirty)

For one-off "run this in background":

```bash
long-running-thing &              # & = background; dies when shell exits
nohup long-running-thing &        # survives shell exit, logs to nohup.out

jobs                              # list background jobs in this shell
fg %1                             # bring job 1 to foreground
bg %1                             # resume in background
disown %1                         # detach from shell entirely
```

### systemd: The Production Way

`systemd` is the **service manager** on every modern Linux distro. It starts services on boot, restarts them on crash, captures their logs, and handles dependencies.

#### Anatomy of a `.service` file

Create `/etc/systemd/system/myapp.service`:

```ini
[Unit]
Description=My DevOps App
After=network.target

[Service]
Type=simple
User=sumit
WorkingDirectory=/home/sumit/myapp
ExecStart=/usr/bin/python3 /home/sumit/myapp/server.py
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
Environment="PORT=8000"
EnvironmentFile=-/etc/myapp.env

[Install]
WantedBy=multi-user.target
```


| Section     | Key                               | Meaning                                 |
| ----------- | --------------------------------- | --------------------------------------- |
| `[Unit]`    | `Description`                     | Human-readable name                     |
| `[Unit]`    | `After=network.target`            | Wait until network is up                |
| `[Service]` | `Type=simple`                     | Default: process stays in foreground    |
| `[Service]` | `Restart=on-failure`              | Auto-restart on crash                   |
| `[Service]` | `EnvironmentFile=-/etc/myapp.env` | Load env vars (`-` = ignore if missing) |
| `[Install]` | `WantedBy=multi-user.target`      | Start on boot at the "multi-user" stage |


#### Service Lifecycle Commands

```bash
sudo systemctl daemon-reload          # tell systemd to re-read unit files
sudo systemctl start myapp.service
sudo systemctl stop myapp.service
sudo systemctl restart myapp.service
sudo systemctl status myapp.service
sudo systemctl enable myapp.service   # start on boot
sudo systemctl disable myapp.service
```

#### Viewing Logs with `journalctl`

```bash
journalctl -u myapp.service           # all logs from myapp
journalctl -u myapp.service -f        # tail (live)
journalctl -u myapp.service --since "1 hour ago"
journalctl -u myapp.service -n 50     # last 50 lines
journalctl -u myapp.service -p err    # only errors
```

#### systemd Timers (Better Cron)

Create `/etc/systemd/system/healthcheck.service`:

```ini
[Unit]
Description=Health Check
[Service]
Type=oneshot
ExecStart=/home/sumit/scripts/healthcheck.sh
```

Create `/etc/systemd/system/healthcheck.timer`:

```ini
[Unit]
Description=Run health check every 5 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=5min
Unit=healthcheck.service

[Install]
WantedBy=timers.target
```

Enable + start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now healthcheck.timer
systemctl list-timers
```

Why timers over cron?

- Native log integration (`journalctl`)
- Better dependency handling
- Run on boot if missed
- Visible in `systemctl status`

---

## Module 4 — Networking Fundamentals

### TCP vs UDP


|             | TCP                 | UDP                         |
| ----------- | ------------------- | --------------------------- |
| Connection  | Yes (handshake)     | No                          |
| Reliability | Guarantees delivery | Best effort                 |
| Order       | Preserved           | Not guaranteed              |
| Speed       | Slower              | Faster                      |
| Used for    | HTTP, SSH, DBs      | DNS, video streaming, games |


### Ports & Sockets

A **port** is a 16-bit number (0–65535). A **socket** is `(IP, port, protocol)`.


| Port      | Service          |
| --------- | ---------------- |
| 22        | SSH              |
| 53        | DNS              |
| 80        | HTTP             |
| 443       | HTTPS            |
| 3306      | MySQL            |
| 5432      | Postgres         |
| 6379      | Redis            |
| 8000–9000 | Common dev ports |


Ports < 1024 are "privileged" — only root can bind to them.

### SSH & Key-Based Auth

Generate a key pair:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
# Press enter to accept default location (~/.ssh/id_ed25519)
# Set a passphrase (recommended)
```

You now have two files:

- `~/.ssh/id_ed25519` — **private key** (never share)
- `~/.ssh/id_ed25519.pub` — **public key** (share freely)

Copy your public key to a remote server:

```bash
ssh-copy-id sumit@my-server.com
# Or manually: append the .pub to ~/.ssh/authorized_keys on the server
```

SSH config (`~/.ssh/config`) for convenience:

```
Host myserver
    HostName 203.0.113.42
    User sumit
    IdentityFile ~/.ssh/id_ed25519
    Port 22
```

Now: `ssh myserver` just works.

### Diagnostic Tools

#### `ping` — is the host reachable?

```bash
ping -c 4 google.com
# PING google.com (142.250.74.46) 56(84) bytes of data.
# 64 bytes from ... time=12.3 ms
# 64 bytes from ... time=11.9 ms
# ...
```

#### `curl` — make HTTP requests

```bash
curl https://httpbin.org/get                          # GET
curl -v https://example.com                           # verbose (show headers)
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"sumit"}' https://api.example.com    # POST with JSON
curl -o file.zip https://example.com/file.zip         # save to file
curl -I https://google.com                            # HEAD (just headers)
```

#### `dig` — DNS lookups

```bash
dig google.com
# ;; ANSWER SECTION:
# google.com.         300     IN      A       142.250.74.46

dig google.com MX                          # mail exchanger records
dig +short google.com                      # just the answer
dig @1.1.1.1 google.com                    # query specific resolver
```

#### `ss` — what's listening?

```bash
sudo ss -tlnp                              # TCP, listening, numeric, with PID
# State   Recv-Q  Send-Q   Local Address:Port     Process
# LISTEN  0       4096           *:22             users:(("sshd",pid=1234,fd=3))
# LISTEN  0       128            *:80             users:(("nginx",pid=5678,fd=6))
```

#### `traceroute` — the network path

```bash
traceroute google.com
```

### DNS Deep Dive

DNS resolves human names to IPs.

```
yourbrowser
    ↓ "what's the IP for example.com?"
recursive resolver (8.8.8.8 / 1.1.1.1)
    ↓
root DNS (.) → TLD DNS (com.) → authoritative DNS (example.com.)
    ↓
93.184.216.34
```

**Common record types:**


| Type    | Purpose                                         | Example                                               |
| ------- | ----------------------------------------------- | ----------------------------------------------------- |
| `A`     | IPv4 address                                    | `example.com → 93.184.216.34`                         |
| `AAAA`  | IPv6 address                                    | `example.com → 2606:2800:220:1::`                     |
| `CNAME` | Alias to another name                           | `www.example.com → example.com`                       |
| `MX`    | Mail server                                     | `example.com → mail.example.com priority 10`          |
| `TXT`   | Arbitrary text (SPF, DKIM, domain verification) | `example.com → "v=spf1 include:_spf.google.com ~all"` |
| `NS`    | Authoritative nameservers                       | `example.com → ns1.cloudflare.com`                    |


### HTTP Basics

Methods:

- `GET` — fetch a resource
- `POST` — create
- `PUT` — replace
- `PATCH` — modify
- `DELETE` — delete

Status codes:

- `2xx` — success (`200 OK`, `201 Created`, `204 No Content`)
- `3xx` — redirect (`301 Moved Permanently`, `302 Found`)
- `4xx` — client error (`400 Bad Request`, `401 Unauthorized`, `404 Not Found`)
- `5xx` — server error (`500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`)

Headers:

- `Content-Type: application/json`
- `Authorization: Bearer <token>`
- `User-Agent: Mozilla/5.0 ...`
- `Cache-Control: no-cache`

---

## Module 5 — Web Servers, Reverse Proxies & TLS

### Forward Proxy vs Reverse Proxy

```
Forward Proxy (acts FOR client):
[Client] → [Forward Proxy] → [Server]
                ↑
        Hides client identity. Used for filtering, caching.
        Example: corporate proxy, Tor.

Reverse Proxy (acts FOR server):
[Client] → [Reverse Proxy] → [Backend Servers]
                ↑
        Hides backend topology. Adds TLS, LB, caching.
        Example: nginx in front of your app.
```

### nginx: The Workhorse

Install:

```bash
sudo apt install nginx
sudo systemctl enable --now nginx
sudo systemctl status nginx
# verify in browser: http://localhost/
```

Config structure:

- `/etc/nginx/nginx.conf` — main config
- `/etc/nginx/sites-available/` — your site configs (write here)
- `/etc/nginx/sites-enabled/` — symlinks to active sites
- `/var/log/nginx/access.log`, `error.log` — logs

#### Example 1: Static file server

Create `/etc/nginx/sites-available/mysite`:

```nginx
server {
    listen 80;
    server_name mysite.local;
    root /var/www/mysite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable + reload:

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo nginx -t          # syntax check — ALWAYS do this before reload
sudo systemctl reload nginx
```

#### Example 2: Reverse proxy to a backend app

Your Flask app runs on `127.0.0.1:8000`. You want users to hit `http://api.example.com` and have nginx forward to it.

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Reload and test:

```bash
sudo nginx -t && sudo systemctl reload nginx
curl http://api.example.com/health
# whatever your backend returns
```

#### Example 3: Load balancing across multiple backends

```nginx
upstream backend {
    server 10.0.0.10:8000;
    server 10.0.0.11:8000;
    server 10.0.0.12:8000;
    # least_conn;        # use least connections
    # ip_hash;           # sticky by client IP
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

### TLS / HTTPS with Let's Encrypt

Free 90-day certificates. Certbot automates renewal.

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d api.example.com
# follow prompts
```

Certbot will:

1. Prove you own the domain (HTTP-01 challenge)
2. Get a cert from Let's Encrypt
3. **Modify your nginx config** to use it
4. Set up auto-renewal via systemd timer

Verify renewal works:

```bash
sudo certbot renew --dry-run
```

After Certbot, your nginx config looks like:

```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8000;
    }
}

server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;   # redirect HTTP → HTTPS
}
```

### Apache (Brief Comparison)


|              | nginx                          | Apache                     |
| ------------ | ------------------------------ | -------------------------- |
| Architecture | Event-driven (async)           | Process/thread per request |
| Config       | One big tree                   | `.htaccess` per directory  |
| Memory       | Lower                          | Higher                     |
| Static files | Faster                         | Slower                     |
| Used in      | Modern stacks, reverse proxies | Legacy stacks, PHP         |


Most modern deployments pick nginx. Apache is still huge in legacy enterprise + PHP shops.

### Caddy (Mention)

A newer web server that handles HTTPS automatically (no certbot setup). Great for hobby projects:

```caddyfile
api.example.com {
    reverse_proxy 127.0.0.1:8000
}
```

That's the entire config. Caddy gets the cert, renews it, redirects HTTP→HTTPS, all out of the box.

---

## Module 6 — Git & GitHub Mastery

### The Three Areas

```
[Working Tree] → git add → [Staging Area] → git commit → [Repository]
     ↑                                                          ↓
     └──────────────────── git checkout ───────────────────────┘
```

### Daily Commands

```bash
git init                              # new repo
git clone <url>                       # clone existing
git status                            # what's changed
git diff                              # unstaged changes
git diff --staged                     # staged changes
git add file.txt                      # stage
git add .                             # stage everything
git commit -m "fix: handle empty input"
git log --oneline --graph --all       # pretty history
git push origin main
git pull --rebase origin main         # pull and replay your commits on top
```

### Branching

```bash
git branch                            # list branches
git branch feature-x                  # create
git checkout feature-x                # switch
git checkout -b feature-x             # create + switch (shorthand)
git switch feature-x                  # modern alternative
git branch -d feature-x               # delete (safe)
git branch -D feature-x               # delete (force)
```

### Merge vs Rebase

**Merge** creates a merge commit, preserves history:

```bash
git checkout main
git merge feature-x
```

```
*   merge commit
|\
| * feature commit 2
| * feature commit 1
* | main commit
|/
* common ancestor
```

**Rebase** replays your commits on top of the target — linear history:

```bash
git checkout feature-x
git rebase main
```

```
* feature commit 2 (replayed)
* feature commit 1 (replayed)
* main commit
* common ancestor
```

**Rule of thumb:** rebase your local feature branches; never rebase shared branches.

### Resolving Conflicts

When Git can't auto-merge, you get conflict markers:

```
<<<<<<< HEAD
their version
=======
your version
>>>>>>> feature-x
```

Edit the file, remove markers, then:

```bash
git add file.txt
git commit                # finishes the merge/rebase
```

If you're rebasing and want to abort:

```bash
git rebase --abort
```

### `.gitignore`

```
# environment files
.env
*.env.local

# build artifacts
node_modules/
__pycache__/
*.pyc
dist/
build/

# IDE
.vscode/
.idea/

# OS junk
.DS_Store
Thumbs.db
```

### Lifesaving Commands

```bash
git stash                  # save uncommitted changes for later
git stash pop              # restore them
git reflog                 # see every HEAD movement (recover lost commits!)
git bisect start           # binary search for a bad commit
git cherry-pick abc123     # apply one commit to current branch
```

### GitHub Workflow (PR-driven)

```bash
# Fork repo on GitHub, then:
git clone https://github.com/YOU/repo.git
cd repo
git remote add upstream https://github.com/ORIGINAL/repo.git

# Create a branch
git checkout -b fix/login-bug
# ... edit files ...
git add .
git commit -m "fix: redirect after login"
git push -u origin fix/login-bug

# Open a PR on GitHub UI, or:
gh pr create --title "Fix login redirect" --body "Fixes #42"
```

### Branching Strategies

**Git Flow** — `main`, `develop`, `feature/`*, `release/`*, `hotfix/*`. Heavyweight. Used by big enterprise teams.

**GitHub Flow** — `main` is always deployable; feature branches off main; PR → merge → deploy. Simple. Used by most modern teams.

**Trunk-Based Development** — everyone commits to `main` (or short-lived branches < 1 day). Requires strong CI + feature flags. Used by Google, Facebook.

**For CSOT projects: use GitHub Flow.**

### Conventional Commits

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`.

Examples:

```
feat(auth): add Google OAuth login
fix(api): handle null user in /profile
docs: update README setup steps
chore(deps): bump nginx to 1.27
```

Conventional Commits enable automatic CHANGELOG and version bumping (Week 2).

---

## Module 7 — Secrets Hygiene

### The #1 Way to Get Hacked

```bash
# DO NOT DO THIS:
git commit -m "added API key"
git push origin main
# Within minutes, bots scan GitHub for AWS keys → drain your account.
```

### Rules

1. **Never commit secrets.** Not even temporarily. Not even to a private repo.
2. **Use `.env` files**, git-ignored, loaded at runtime.
3. **Commit a `.env.example`** with empty/dummy values, documenting required vars.
4. **Scan your history** with TruffleHog or gitleaks.
5. **If you leak, rotate immediately.** Don't try to hide it with `git rebase` — assume the world saw it.

### `.env` Pattern

```bash
# .env (git-ignored!)
DATABASE_URL=postgres://user:realpassword@db.example.com/myapp
GEMINI_API_KEY=AIzaSyD_xxxxxxxxxxxxxxxxxxxxxxxxx
SECRET_KEY=a-long-random-string

# .env.example (committed)
DATABASE_URL=postgres://user:password@host/db
GEMINI_API_KEY=your-key-here
SECRET_KEY=generate-with-openssl-rand
```

`.gitignore`:

```
.env
.env.local
.env.*.local
```

Loading in Python:

```python
from dotenv import load_dotenv
import os

load_dotenv()
db_url = os.environ["DATABASE_URL"]
```

Loading in Node:

```javascript
require('dotenv').config();
const dbUrl = process.env.DATABASE_URL;
```

Loading in Bash:

```bash
set -a; source .env; set +a
```

### TruffleHog: Scanning for Leaks

Install:

```bash
# Linux/Mac:
curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh \
    | sh -s -- -b /usr/local/bin

# Or with Go: go install github.com/trufflesecurity/trufflehog/v3@latest
# Or with brew: brew install trufflehog
```

Scan a repo's full history (including deleted commits!):

```bash
trufflehog git file://. --only-verified
```

Sample output if clean:

```
🐷🔑 TruffleHog. Unearth your secrets. 🔑🐷
🐷 Scan completed.
```

Sample output if leaked:

```
Found verified result 🐷🔑
Detector Type: AWS
Decoder Type: PLAIN
Raw result: AKIAIOSFODNN7EXAMPLE
File: src/config.py
Commit: a1b2c3d4
Email: sumit@example.com
```

### Pre-commit Hook (Catches Secrets BEFORE Commit)

```bash
pip install pre-commit
```

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/trufflesecurity/trufflehog
    rev: v3.63.7
    hooks:
      - id: trufflehog
        name: TruffleHog
        entry: bash -c 'trufflehog git file://. --since-commit HEAD --only-verified --fail'
        language: system
        stages: ["commit"]
```

Install:

```bash
pre-commit install
```

Now every `git commit` runs the scanner first.

### What to Do If You Leak

1. **Rotate the secret immediately** (AWS console → delete key → make new one).
2. **Don't `git rebase` and force-push** — the commit is in GitHub's caches; assume it's seen.
3. **Use BFG Repo-Cleaner** to scrub history (optional, only for very embarrassing leaks):
  ```bash
   bfg --delete-files .env
   git reflog expire --expire=now --all && git gc --prune=now --aggressive
   git push --force
  ```
4. **Audit access logs** for misuse.

---

## Build 1 — System Health Monitor (Wed)

### Goal

A shell script that captures system health (disk, memory, CPU, top processes, network interfaces), runs every 5 minutes via a `systemd` timer, and logs to `journalctl`.

### Step 1: Write the Script

Save as `~/scripts/healthmon.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

ts=$(date -Iseconds)

echo "=== HEALTH CHECK $ts ==="

echo "[hostname] $(hostname)"
echo "[uptime] $(uptime -p)"

echo "[load]"
cat /proc/loadavg

echo "[memory]"
free -h | grep -E '^Mem|^Swap'

echo "[disk]"
df -h --output=target,size,used,avail,pcent | grep -vE '^Filesystem|^tmpfs|^devtmpfs|^/dev/loop'

echo "[top-5-cpu]"
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -6

echo "[network]"
ip -brief addr | grep -v 'DOWN'

echo "=== END $ts ==="
```

Make it executable:

```bash
chmod +x ~/scripts/healthmon.sh
~/scripts/healthmon.sh        # test it
```

Expected output:

```
=== HEALTH CHECK 2026-05-25T10:30:00+05:30 ===
[hostname] sumit-laptop
[uptime] up 2 days, 4 hours
[load]
0.42 0.55 0.61 2/812 12345
[memory]
Mem:           15Gi       4.2Gi       8.9Gi       312Mi       2.1Gi        10Gi
Swap:         2.0Gi          0B       2.0Gi
[disk]
Mounted on        Size  Used Avail Use%
/                 234G   84G  138G  38%
/boot             511M   53M  458M  11%
/home             450G  120G  308G  29%
[top-5-cpu]
    PID USER     %CPU %MEM CMD
   1234 sumit    12.3  2.1 chrome --type=renderer ...
   5678 sumit     8.7  4.5 code ...
   ...
[network]
lo               UNKNOWN        127.0.0.1/8 ::1/128
wlan0            UP             192.168.1.42/24
=== END 2026-05-25T10:30:00+05:30 ===
```

### Step 2: Create the systemd Service

`/etc/systemd/system/healthmon.service`:

```ini
[Unit]
Description=System Health Monitor (one-shot)
[Service]
Type=oneshot
ExecStart=/home/sumit/scripts/healthmon.sh
User=sumit
StandardOutput=journal
StandardError=journal
```

### Step 3: Create the Timer

`/etc/systemd/system/healthmon.timer`:

```ini
[Unit]
Description=Run healthmon every 5 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=5min
Unit=healthmon.service

[Install]
WantedBy=timers.target
```

### Step 4: Enable

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now healthmon.timer
systemctl list-timers | grep healthmon
```

Expected:

```
NEXT                        LEFT     LAST    PASSED   UNIT             ACTIVATES
Mon 2026-05-25 10:35:00 IST 3min ago 2min    healthmon.timer  healthmon.service
```

### Step 5: Verify

```bash
journalctl -u healthmon.service -f
```

You'll see new output every 5 minutes. 

### ✅ Build 1 Complete When:

- `~/scripts/healthmon.sh` is executable and produces sensible output
- `healthmon.timer` is **active** (`systemctl status healthmon.timer`)
- `journalctl -u healthmon.service` shows runs every 5 min
- Screenshot of `journalctl` output committed to your repo

---

## Build 2 — App Behind nginx with HTTPS (Fri)

### Goal

Run a small web app under systemd, fronted by nginx as a reverse proxy on ports 80 and 443 with HTTPS.

### Step 1: A Tiny App

Save as `~/myapp/server.py`:

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import os, json, datetime

class H(BaseHTTPRequestHandler):
    def do_GET(self):
        body = json.dumps({
            "service": "csot-demo",
            "time": datetime.datetime.now().isoformat(),
            "path": self.path,
        }).encode()
        self.send_response(200)
        self.send_header("Content-Type", "application/json")
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)
    def log_message(self, fmt, *args):
        print(f"{self.address_string()} - {fmt%args}", flush=True)

if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    print(f"listening on :{port}", flush=True)
    HTTPServer(("127.0.0.1", port), H).serve_forever()
```

Test:

```bash
python3 ~/myapp/server.py &
curl http://127.0.0.1:8000/hello
# {"service": "csot-demo", "time": "...", "path": "/hello"}
kill %1
```

### Step 2: systemd Service for the App

`/etc/systemd/system/myapp.service`:

```ini
[Unit]
Description=CSOT Demo App
After=network.target

[Service]
Type=simple
User=sumit
WorkingDirectory=/home/sumit/myapp
ExecStart=/usr/bin/python3 /home/sumit/myapp/server.py
Restart=on-failure
RestartSec=3
Environment=PORT=8000
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
sudo systemctl status myapp.service
curl http://127.0.0.1:8000/
```

### Step 3: nginx as Reverse Proxy

`/etc/nginx/sites-available/myapp`:

```nginx
server {
    listen 80;
    server_name api.yourdomain.com;   # change to your domain

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /health {
        access_log off;
        return 200 "ok\n";
    }
}
```

Enable + test:

```bash
sudo ln -sf /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/myapp
sudo nginx -t
sudo systemctl reload nginx

curl http://api.yourdomain.com/
# {"service": "csot-demo", ...}
```

### Step 4: HTTPS with Let's Encrypt

DNS must already point your domain to this server.

```bash
sudo certbot --nginx -d api.yourdomain.com
# Choose: 2 (redirect HTTP → HTTPS)
```

Verify:

```bash
curl https://api.yourdomain.com/hello
# {"service": "csot-demo", "path": "/hello", ...}

curl -I http://api.yourdomain.com/
# HTTP/1.1 301 Moved Permanently
# Location: https://api.yourdomain.com/
```

### Don't Have a Domain? Use Cloudflare Tunnel

```bash
# Install cloudflared (Linux):
curl -L --output cloudflared.deb \
   https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb

cloudflared tunnel login                              # opens browser, log in
cloudflared tunnel create csot-demo
cloudflared tunnel route dns csot-demo demo.yourname.dev
cloudflared tunnel run --url http://127.0.0.1:8000 csot-demo
```

Cloudflare gives you free HTTPS, no certbot needed.

### ✅ Build 2 Complete When:

- App runs under `systemd` (survives logout, restarts on crash)
- nginx reverse-proxies port 80 → 8000
- HTTPS works on port 443 (Let's Encrypt or Cloudflare)
- HTTP → HTTPS redirect in place
- Screenshot of `curl https://...` from a **different machine** committed to repo

---

## Weekly Mini-Project — DevOps Toolkit Repo

> **Submission deadline: Sunday 11:59 PM (IST)**

This project has two tracks. 

### 🟢 Part A — Local Track

A GitHub repo combining Build 1 + Build 2 + extra polished shell utilities into a portfolio piece. **Runs entirely on your local Linux/WSL/Mac.** 

#### Required Contents


| File                      | Purpose                                                                         |
| ------------------------- | ------------------------------------------------------------------------------- |
| `README.md`               | Project intro, usage, demo screenshots                                          |
| `DEMO.md`                 | Sample outputs of every script                                                  |
| `.gitignore`              | Excludes `.env`, build artifacts, OS junk                                       |
| `.env.example`            | Template env file                                                               |
| `LICENSE`                 | MIT recommended                                                                 |
| `scripts/backup.sh`       | Backs up a directory to `.tar.gz` with timestamp; logs run                      |
| `scripts/log_parser.sh`   | Parses nginx/Apache logs → top IPs, 4xx/5xx counts, top URL                     |
| `scripts/user_manager.sh` | Creates/deletes Linux users from a CSV (safe to test in container)              |
| `scripts/sysreport.sh`    | Build 1's health monitor                                                        |
| `scripts/deploy.sh`       | Deploys Build 2 (app + nginx + systemd) to a fresh Linux box / Docker container |
| `systemd/`                | Unit files for Build 1's timer and Build 2's service                            |
| `nginx/`                  | Build 2's nginx site config                                                     |


#### How to Demo HTTPS Without a Real Domain

Pick whichever is easier:

**Option 1 — Self-signed cert (works on `localhost`):**

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/myapp.key -out /etc/ssl/certs/myapp.crt \
  -subj "/CN=localhost"

# Reference these in your nginx site config:
#   ssl_certificate     /etc/ssl/certs/myapp.crt;
#   ssl_certificate_key /etc/ssl/private/myapp.key;

curl -k https://localhost/   # -k skips cert verification
```

**Option 2 — Free Cloudflare Quick Tunnel (real HTTPS, real URL):**

```bash
cloudflared tunnel --url http://localhost:8000
# https://chunky-bear-xyz.trycloudflare.com
```

No domain, no account, no cost. The URL is throwaway but real HTTPS.

Either is acceptable for grading.

### Example: `scripts/backup.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

usage() { echo "Usage: $0 <source-dir> <dest-dir>"; exit 1; }
[[ $# -ne 2 ]] && usage

src="$1"
dest="$2"
[[ ! -d "$src" ]] && { echo "Source dir not found: $src" >&2; exit 1; }
mkdir -p "$dest"

ts=$(date +%Y%m%d-%H%M%S)
out="$dest/$(basename "$src")-$ts.tar.gz"

tar czf "$out" -C "$(dirname "$src")" "$(basename "$src")"
size=$(du -h "$out" | cut -f1)

echo "[$ts] backed up $src → $out ($size)" | tee -a "$dest/backup.log"
```

Usage:

```bash
./scripts/backup.sh ~/Documents /tmp/backups
# [20260525-103200] backed up /home/sumit/Documents → /tmp/backups/Documents-20260525-103200.tar.gz (842M)
```

### Example: `scripts/log_parser.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

usage() { echo "Usage: $0 <access.log>"; exit 1; }
[[ $# -ne 1 ]] && usage
log="$1"
[[ ! -f "$log" ]] && { echo "Log not found: $log" >&2; exit 1; }

echo "=== Top 10 IPs ==="
awk '{print $1}' "$log" | sort | uniq -c | sort -rn | head -10

echo
echo "=== HTTP Status Distribution ==="
awk '{print $9}' "$log" | sort | uniq -c | sort -rn

echo
echo "=== Top 10 URLs ==="
awk '{print $7}' "$log" | sort | uniq -c | sort -rn | head -10

echo
echo "=== 4xx Errors ==="
awk '$9 ~ /^4/' "$log" | wc -l

echo "=== 5xx Errors ==="
awk '$9 ~ /^5/' "$log" | wc -l
```

#### Submission Checklist (Part A)

- Public GitHub repo
- All 5 scripts work (run `./script.sh --help` or with sample input → meaningful output)
- Build 1 systemd timer is set up and running (screenshot in DEMO.md)
- Build 2 nginx works on `localhost` with either self-signed HTTPS **or** Cloudflare Tunnel
- Each script handles missing args, empty input, file-not-found
- `.env.example` committed; `.env` git-ignored
- Commit history: ≥ 4 commits across 2+ branches with one merge
- `**trufflehog git file://. --only-verified` returns clean**

#### Grading Rubric — Part A (50 pts)


| Criterion                                                        | Points |
| ---------------------------------------------------------------- | ------ |
| All 5 scripts functional                                         | 15     |
| Build 2's nginx + HTTPS (any flavor) works                       | 10     |
| Build 1's systemd timer configured correctly                     | 5      |
| Error handling / edge cases                                      | 5      |
| README + DEMO.md quality                                         | 5      |
| Git hygiene (branches, commits, merge)                           | 5      |
| **TruffleHog clean** (auto-fail if any leaked secret in history) | 5      |


---

## Alternative Mini-Project Ideas (Optional) 

Pick one of these instead of the default.

### 1. Hardened Personal Server

Take a free-tier VM (Oracle ARM, AWS t2.micro, GCP e2-micro). Harden SSH (key-only, disable root login, change port, `fail2ban`). Install nginx + HTTPS via Certbot. Deploy a portfolio HTML page. Write a `systemd` service for an uptime monitor that pings 3 services and logs to journalctl.

**Deliverables**: Repo with all configs + an SSH-test report proving lockdown.

### 2. Self-Hosted Status Page

Pure bash + nginx project. systemd timer pings 5 services every minute, generates a static HTML status page served by nginx. Bonus: Discord/Slack webhook on outage.

**Deliverables**: Repo + live URL.

### 3. Auto-Backup System

`rsync` over SSH to a remote host on a schedule (systemd timer), with rotation (keep last 7 daily, 4 weekly, 12 monthly) and email/Discord notification on failure.

**Deliverables**: Repo + journalctl output showing successful runs.

### 4. Log Analytics CLI

Pure-shell tool that parses nginx/Apache logs into a clean Markdown or HTML daily report. Supports filtering by date range, IP range, status code. Outputs charts (using ASCII or `gnuplot`).

**Deliverables**: Repo + sample reports.

### 5. `devbox`-Style Dotfiles Manager

A shell tool that bootstraps a new Linux machine with your tools, configs, and dotfiles in one command. Idempotent. Uses Git for the dotfiles.

**Deliverables**: Repo + tested on fresh Docker container.

### 6. (AI Track) LLM Rate-Limit Logger

nginx as reverse proxy in front of a free LLM API (Gemini/Groq). Shell script parses access logs to track token usage and rate limits per minute. Outputs a daily report. systemd timer.

**Deliverables**: Repo + daily report sample + nginx config.

## Contest reference — tasks, scoring, rules

> **Mandatory · 100 points · runs the full week** — see [Week 1 Contest](#week-1-contest--mandatory-100-points) above for the requirement and quick start.
>
> **Platform: [csot-devops.devclub.in](https://csot-devops.devclub.in)** &middot; sign in with your DevClub account. Exact open/close timestamps are shown on the site when you sign in.

**12 tasks · 100 points total.** Autograded in a locked-down Docker sandbox. Submit the
whole folder or one task at a time via the `csot` CLI.

**Scoring:** your leaderboard score is the **sum of your best mark on each task** across all attempts (not the best single upload total). Many tasks award **partial credit** step-by-step. On several tasks the grader splits checks roughly **50% visible** (you see expected vs got on failure) and **50% hidden** (pass/fail only). **Penalties do not subtract points** — they break ties only, and stop after your first full submit without `-t`.

Live **overall** and **weekly** leaderboards: [csot-devops.devclub.in/leaderboard](https://csot-devops.devclub.in/leaderboard).

📖 **Full task specs, skeleton examples, and grader rules:**
**[csot-devops.devclub.in/contest/week-01](https://csot-devops.devclub.in/contest/week-01)** ←
*everything you need is there; this section is the elevator pitch.*

### 60-second start

```bash
# install + sign-in (one-time)
curl -fsSL https://csot-devops.devclub.in/install.sh | bash
csot login                              # paste-code flow via DevClub

# write your solutions, then submit
csot submit ./my-solutions              # all 12 tasks
csot submit ./my-solutions -t 07        # only task 07
csot submit ./07.service  -t 07         # single-file submit

# iterate
csot history                            # your past attempts
csot show <attempt-id>                  # full per-task pass/fail
csot leaderboard                        # default: Overall
csot leaderboard --weekly               # this week
csot update                             # pull latest CLI
```

### The 12 tasks (summary — full specs on the site)


| #   | What                                              | Submit as                 | Pts | Partial? |
| --- | ------------------------------------------------- | ------------------------- | --- | -------- |
| 1   | Find files larger than 1 MiB, sorted desc by size | `01.sh`                   | 7   | yes      |
| 2   | Count unique IPv4 addresses in a log              | `02.sh`                   | 7   | yes      |
| 3   | Replace tabs with 4 spaces, recursively in place  | `03.sh`                   | 7   | —        |
| 4   | Rename `*.txt` → `*.md` recursively               | `04.sh`                   | 7   | —        |
| 5   | Parse JSON with `jq` (active users' emails)       | `05.sh`                   | 9   | yes      |
| 6   | Retry a command with exponential backoff          | `06.sh`                   | 10  | yes      |
| 7   | systemd timer running `Hello World` every minute  | `07.service` + `07.timer` | 9   | yes      |
| 8   | nginx reverse-proxy config + `X-Powered-By: csot` | `08.conf`                 | 9   | yes      |
| 9   | Self-signed cert + nginx HTTPS on `:8443`         | `09.sh`                   | 10  | yes      |
| 10  | Top 10 directories by disk usage                  | `10.sh`                   | 8   | —        |
| 11  | List all running systemd services                 | `11.sh`                   | 7   | —        |
| 12  | `todo` CLI in pure bash (capstone)                | `12-todo/todo.sh`         | 10  | yes      |


**Total: 100 points.**

Each task page lists **Submit as**, **Input**, **Output**, skeleton examples (not full solutions), visible vs hidden checks where applicable, and hints. **Visit the site — not a black box:** [csot-devops.devclub.in/contest/week-01](https://csot-devops.devclub.in/contest/week-01).

### Rules at a glance

- **Window**: full Week 1 (open all week; exact times on the contest site)
- **Score** = sum of per-task bests across attempts; partial marks add up within a task
- **Penalties** tie-break only (before first full submit); they do **not** reduce your score
- **Tie-break**: higher score → fewer penalties → fewer attempts → earlier first graded submission
- Identical re-submissions are deduped (no grade, no penalty)
- **Rate limit**: 5/min, 100/contest
- **Submission size**: ≤ 1 MB, ≤ 50 files; per-task timeout 30 s
- **Privacy**: your code is uploaded only to the server; other students never see it
- **Code of honour**: AI/man pages/blog posts fine; copy-pasting a peer's solution is not

---

## Resources

### Books (Free)

- *The Linux Command Line* — William Shotts (free PDF: linuxcommand.org)
- *Pro Git* — Chacon & Straub (git-scm.com/book)
- *The Twelve-Factor App* — 12factor.net

### Videos

- TechWorld with Nana — Linux Crash Course
- DevOps Toolkit — nginx Tutorial
- Computerphile — How DNS Works

### Cheatsheets

- `tldr` command (`sudo apt install tldr` then `tldr nginx`)
- DevHints.io (lots of one-pagers)
- nginx Beginner's Guide (nginx.org)

### Tutorials

- DigitalOcean — *How To Install nginx on Ubuntu*
- DigitalOcean — *Secure nginx with Let's Encrypt*
- DigitalOcean — *Initial Server Setup with Ubuntu*

---

**Next week: Week 2 — CI/CD, Quality Engineering & Registries.** You'll take the Build 2 app and add a full GitHub Actions pipeline (lint → test → schema check → TruffleHog → build → push to GHCR).