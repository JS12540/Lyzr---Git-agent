# How to Run, Test, Push & Register the 3 GitAgents

> Complete guide covering every provider (Groq, OpenAI, Claude, OpenClaw),
> local testing, GitHub setup, and registry submission.

---

## Table of Contents

1. [How GitAgent Works](#1-how-gitagent-works)
2. [Prerequisites](#2-prerequisites)
3. [Agent Overview](#3-agent-overview)
4. [Run with Claude Code](#4-run-with-claude-code)
5. [Run with OpenAI](#5-run-with-openai)
6. [Run with Groq](#6-run-with-groq)
7. [Run with OpenClaw](#7-run-with-openclaw)
8. [Testing Your Agents Locally](#8-testing-your-agents-locally)
9. [Push to GitHub](#9-push-to-github)
10. [Register on the GitAgent Registry](#10-register-on-the-gitagent-registry)
11. [Quick Reference Card](#11-quick-reference-card)

---

## 1. How GitAgent Works

GitAgent is an **open standard** — your git repository IS the agent.
No platform. No proprietary dashboard. Just files.

```
agent-folder/
├── agent.yaml   ← who the agent is (model, skills, config)
├── SOUL.md      ← agent's personality, expertise, communication style
├── RULES.md     ← what it must always / never do
└── skills/
    └── skill-name/
        └── SKILL.md  ← instructions for a specific capability
```

When you run `npx @open-gitagent/gitagent@latest run`, the CLI:

```
1. Reads agent.yaml        → knows the model, skills, runtime config
2. Reads SOUL.md           → builds the system prompt (agent identity)
3. Reads RULES.md          → appends behavioral constraints
4. Reads skills/*/SKILL.md → appends skill instructions
5. Exports to the adapter  → transforms it to the target framework format
6. Launches the agent      → hands off to Claude Code / OpenAI / etc.
```

The adapter is the only thing that changes. The agent definition stays identical.

```
                    ┌─────────────────┐
                    │   agent.yaml    │
                    │   SOUL.md       │  ← Same files for ALL adapters
                    │   RULES.md      │
                    │   skills/       │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────────┐
              ▼              ▼                   ▼
        Claude Code        OpenAI           OpenClaw
        (CLAUDE.md)     (Agents SDK)    (AGENTS.md format)
```

---

## 2. Prerequisites

### Required for all methods

```bash
# Node.js 18+ (check version)
node --version

# Install the gitagent CLI globally
npm install -g @open-gitagent/gitagent

# Confirm installation
gitagent --version
```

### Optional but recommended

```bash
# GitHub CLI — for pushing repos and PRs
# Install: https://cli.github.com
gh --version

# git
git --version
```

---

## 3. Agent Overview

| Agent | Category | Skills | Best For |
|-------|----------|--------|---------|
| `security-audit-agent` | Security | owasp-scanner, secrets-detector, dep-auditor | Code security reviews, CVE scanning |
| `incident-commander-agent` | DevOps | log-analyzer, runbook-writer, postmortem-generator | Production incidents, on-call response |
| `api-architect-agent` | Developer Tools | openapi-designer, endpoint-reviewer, api-doc-writer | API design, OpenAPI specs, docs |

---

## 4. Run with Claude Code

**Best for:** Interactive, multi-turn conversations. Claude Code is the default
and most capable adapter. Fully interactive REPL.

### Setup

```bash
# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Authenticate (one time)
claude
# Follow the browser login flow
```

### Run any agent (one-liner, no clone needed)

```bash
# Security Audit Agent
npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/security-audit-agent \
  -a claude

# Incident Commander Agent
npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/incident-commander-agent \
  -a claude

# API Architect Agent
npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/api-architect-agent \
  -a claude
```

### Run from local folder (before pushing to GitHub)

```bash
# From the git_agent/agents/ directory
gitagent run -d ./security-audit-agent -a claude
gitagent run -d ./incident-commander-agent -a claude
gitagent run -d ./api-architect-agent -a claude
```

### Run with a one-shot prompt (no interactive session)

```bash
gitagent run -d ./security-audit-agent -a claude \
  -p "Audit this code for vulnerabilities: def login(user, pwd): query = 'SELECT * FROM users WHERE user=' + user"

gitagent run -d ./api-architect-agent -a claude \
  -p "Design a REST API for a task management app with users, projects, and tasks"
```

### Useful flags

| Flag | What it does |
|------|-------------|
| `-a claude` | Use Claude Code adapter |
| `-d ./folder` | Run from local directory |
| `-r https://...` | Run from GitHub repo (cached) |
| `-p "prompt"` | One-shot mode — skips interactive REPL |
| `-b develop` | Use a specific branch |
| `--refresh` | Force re-clone (clears cache) |
| `--no-cache` | Don't cache the cloned repo |

---

## 5. Run with OpenAI

**Best for:** OpenAI GPT-4o / GPT-4o-mini. Generates a Python file using
the Agents SDK and runs it.

### Setup

```bash
# Get your key from https://platform.openai.com/api-keys
export OPENAI_API_KEY=sk-your-key-here

# Windows PowerShell
$env:OPENAI_API_KEY="sk-your-key-here"

# Windows CMD
set OPENAI_API_KEY=sk-your-key-here
```

### Run

```bash
# Security Audit Agent
OPENAI_API_KEY=sk-... npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/security-audit-agent \
  -a openai \
  -p "Audit this Python code: import os; query = 'SELECT * FROM users WHERE id = ' + request.args.get('id')"

# Incident Commander Agent
OPENAI_API_KEY=sk-... npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/incident-commander-agent \
  -a openai \
  -p "Analyze these logs and find the root cause: [ERROR] Connection pool exhausted after 847 requests"

# API Architect Agent
OPENAI_API_KEY=sk-... npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/api-architect-agent \
  -a openai \
  -p "Design a REST API for a food delivery platform"
```

### From local folder

```bash
cd agents/security-audit-agent
OPENAI_API_KEY=sk-... gitagent run -d . -a openai -p "Your prompt here"
```

> **Note:** The OpenAI adapter runs in non-interactive (one-shot) mode.
> Always include `-p "prompt"` when using `-a openai`.

---

## 6. Run with Groq

Groq is **not a built-in gitagent adapter** — it's not `claude`, `openai`, etc.
There are two ways to use Groq with these agents.

---

### Method A — GitClaw (Recommended for Groq)

GitClaw is the official git-native runtime from the same team. It supports
Groq natively using the `provider:model` format.

**Step 1 — Install GitClaw**
```bash
# Option A: one-liner
bash <(curl -fsSL "https://raw.githubusercontent.com/open-gitagent/gitclaw/main/install.sh")

# Option B: npm
npm install -g gitclaw
```

**Step 2 — Set your Groq API key**
```bash
# Get free key at: https://console.groq.com → API Keys → Create key

# macOS / Linux
export GROQ_API_KEY=gsk_your_key_here

# Windows PowerShell
$env:GROQ_API_KEY="gsk_your_key_here"

# Windows CMD
set GROQ_API_KEY=gsk_your_key_here
```

**Step 3 — Run**
```bash
# Security Audit Agent
gitclaw --dir ./agents/security-audit-agent \
  --model groq:llama-3.3-70b-versatile \
  "Audit this code for OWASP vulnerabilities and hardcoded secrets"

# Incident Commander Agent
gitclaw --dir ./agents/incident-commander-agent \
  --model groq:llama-3.3-70b-versatile \
  "Analyze these logs: ERROR Connection refused to postgres:5432 after 30s timeout"

# API Architect Agent
gitclaw --dir ./agents/api-architect-agent \
  --model groq:llama-3.3-70b-versatile \
  "Design an OpenAPI 3.1 spec for a SaaS billing API"

# Interactive REPL (no prompt = opens chat)
gitclaw --dir ./agents/security-audit-agent \
  --model groq:llama-3.3-70b-versatile
```

**Available Groq models**

| Model ID | Speed | Quality | Use for |
|----------|-------|---------|---------|
| `llama-3.3-70b-versatile` | Fast | ⭐⭐⭐⭐⭐ | All 3 agents — **recommended** |
| `llama3-70b-8192` | Fast | ⭐⭐⭐⭐ | Good fallback |
| `llama3-8b-8192` | Fastest | ⭐⭐⭐ | Quick tests |
| `gemma2-9b-it` | Fast | ⭐⭐⭐ | Light tasks |

---

### Method B — Export System Prompt + Groq SDK directly

Use when you want to embed the agent in your own Python script.

**Step 1 — Export the system prompt**
```bash
cd agents/security-audit-agent
gitagent export --format system-prompt
# Creates: system_prompt.txt
```

**Step 2 — Call Groq with it**
```python
# pip install groq
import os
from groq import Groq

system_prompt = open("system_prompt.txt").read()

client = Groq(api_key=os.environ["GROQ_API_KEY"])

response = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user",   "content": "Audit this code for SQL injection: " + your_code}
    ],
    temperature=0.1,
    max_tokens=4096,
)

print(response.choices[0].message.content)
```

---

## 7. Run with OpenClaw

**Best for:** Self-hosted, privacy-first, 24/7 persistent agents.
OpenClaw runs a local Gateway on port 18789 and connects to Claude via API.

### Step 1 — Install OpenClaw

```bash
# macOS / Linux
curl -fsSL https://openclaw.ai/install.sh | bash

# Windows (PowerShell as Administrator)
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### Step 2 — Run onboarding

```bash
openclaw onboard --install-daemon
# Follow the prompts:
# 1. Choose model provider → Anthropic (recommended)
# 2. Enter API key → your ANTHROPIC_API_KEY
# 3. Gateway installs and starts on port 18789
```

Get your Anthropic API key at: https://console.anthropic.com → API Keys

```bash
# Verify the gateway is running
openclaw gateway status
# Expected: Gateway running on http://127.0.0.1:18789
```

### Step 3 — Run agents with OpenClaw adapter

```bash
# Security Audit Agent
ANTHROPIC_API_KEY=sk-ant-... npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/security-audit-agent \
  -a openclaw \
  -p "Scan this Python file for OWASP vulnerabilities"

# Incident Commander Agent
ANTHROPIC_API_KEY=sk-ant-... npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/incident-commander-agent \
  -a openclaw \
  -p "Write a runbook for rolling back a Kubernetes deployment"

# API Architect Agent
ANTHROPIC_API_KEY=sk-ant-... npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/api-architect-agent \
  -a openclaw \
  -p "Review this API design for anti-patterns: GET /api/getUser/:id, POST /api/deleteUser"
```

### Step 4 — Access the OpenClaw dashboard

```bash
openclaw dashboard
# Opens http://127.0.0.1:18789 in your browser
# You can chat with deployed agents directly from here
```

> **Note:** OpenClaw uses `ANTHROPIC_API_KEY` (Claude models). It also
> supports OpenAI and Google via provider config in the dashboard settings.

---

## 8. Testing Your Agents Locally

Before pushing to GitHub, test each agent locally with these commands.

### Step 1 — Validate the spec

```bash
# Check each agent conforms to the GitAgent spec
gitagent validate -d ./agents/security-audit-agent
gitagent validate -d ./agents/incident-commander-agent
gitagent validate -d ./agents/api-architect-agent

# Expected output:
# ✅ agent.yaml — valid
# ✅ SOUL.md — present
# ✅ skills/owasp-scanner — valid SKILL.md
# ✅ spec version: 0.1.0
```

### Step 2 — Inspect agent info

```bash
gitagent info -d ./agents/security-audit-agent
# Shows: name, version, model, skills list, SOUL.md preview
```

### Step 3 — Test prompts for each agent

**Security Audit Agent** — paste this code to test:
```bash
gitagent run -d ./agents/security-audit-agent -a claude -p "
Audit this Python code:

import sqlite3

def get_user(user_id):
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()
    query = 'SELECT * FROM users WHERE id = ' + user_id
    cursor.execute(query)
    return cursor.fetchone()

AWS_SECRET = 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY'
DB_PASSWORD = 'admin123'
"
```

**Incident Commander Agent** — test with fake logs:
```bash
gitagent run -d ./agents/incident-commander-agent -a claude -p "
We're getting 503s on our payments service. Here are the logs:

[14:23:01] ERROR QueuePool limit of size 10 overflow 10 reached, connection timed out, timeout 30
[14:23:02] ERROR QueuePool limit of size 10 overflow 10 reached, connection timed out, timeout 30
[14:23:15] ERROR HTTPConnectionPool(host='payments-db', port=5432): Max retries exceeded
[14:23:18] CRITICAL Service payments-api returning 503, error rate 67%

What is the root cause and what do I do right now?
"
```

**API Architect Agent** — test with a design request:
```bash
gitagent run -d ./agents/api-architect-agent -a claude -p "
Design a REST API for a simple blog platform.
Needs: users, posts, comments, tags, and likes.
Output a complete OpenAPI 3.1 spec.
"
```

### Step 4 — Export to different formats and inspect

```bash
cd agents/security-audit-agent

# Export to raw system prompt — shows what the agent will say to the LLM
gitagent export --format system-prompt
cat system_prompt.txt   # inspect the compiled system prompt

# Export to Claude Code format
gitagent export --format claude-code
cat CLAUDE.md           # inspect the CLAUDE.md output

# Export to OpenAI format
gitagent export --format openai
cat agent.py            # inspect the generated Python code
```

---

## 9. Push to GitHub

Each agent needs its own GitHub repository. Do this for all 3.

### Step 1 — Authenticate GitHub CLI (once)

```bash
gh auth login
# Choose: GitHub.com → HTTPS → Login with browser
# Follow the browser auth flow
```

### Step 2 — Push security-audit-agent

```bash
cd agents/security-audit-agent

git init
git add .
git commit -m "feat: initial release v1.0.0"

# Create public GitHub repo and push in one command
gh repo create security-audit-agent \
  --public \
  --description "AI security engineer — OWASP Top 10, secrets detection, CVE scanning" \
  --source=. \
  --push

# Your agent is now live at:
# https://github.com/YOUR_USERNAME/security-audit-agent
```

### Step 3 — Push incident-commander-agent

```bash
cd ../incident-commander-agent

git init
git add .
git commit -m "feat: initial release v1.0.0"

gh repo create incident-commander-agent \
  --public \
  --description "AI SRE for production incidents — log analysis, runbooks, post-mortems" \
  --source=. \
  --push

# https://github.com/YOUR_USERNAME/incident-commander-agent
```

### Step 4 — Push api-architect-agent

```bash
cd ../api-architect-agent

git init
git add .
git commit -m "feat: initial release v1.0.0"

gh repo create api-architect-agent \
  --public \
  --description "AI API architect — OpenAPI 3.1 specs, endpoint review, API docs" \
  --source=. \
  --push

# https://github.com/YOUR_USERNAME/api-architect-agent
```

### Step 5 — Tag a release (required for registry)

```bash
# Do this in each agent directory
git tag v1.0.0
git push origin v1.0.0
```

### Step 6 — Update README run commands

Before registering, update `YOUR_USERNAME` in each `README.md` to your real
GitHub username so the `npx` run command works:

```bash
# Replace YOUR_USERNAME in each README
# In each agent folder, edit README.md:
# Change: https://github.com/YOUR_USERNAME/agent-name
# To:     https://github.com/your-actual-username/agent-name
```

### Verify the agents run from GitHub

```bash
# Test that the run command works end-to-end
npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/security-audit-agent \
  -a claude \
  -p "Hello, what can you do?"
```

---

## 10. Register on the GitAgent Registry

The registry at [registry.gitagent.sh](https://registry.gitagent.sh) is where
agents get discovered. This is the "npm publish" equivalent for GitAgent.

### Prerequisites

- Agent is pushed to a **public** GitHub repository ✅
- `agent.yaml` + `SOUL.md` + `README.md` all present ✅
- `gitagent validate` passes ✅
- `git tag v1.0.0` pushed ✅

### Step 1 — Install gitagent CLI (if not done)

```bash
npm install -g @open-gitagent/gitagent
```

### Step 2 — Register each agent

```bash
# Security Audit Agent
gitagent registry \
  -r https://github.com/YOUR_USERNAME/security-audit-agent \
  -c security \
  -a claude-code,openai,system-prompt

# Incident Commander Agent
gitagent registry \
  -r https://github.com/YOUR_USERNAME/incident-commander-agent \
  -c devops \
  -a claude-code,openai,system-prompt

# API Architect Agent
gitagent registry \
  -r https://github.com/YOUR_USERNAME/api-architect-agent \
  -c developer-tools \
  -a claude-code,openai,system-prompt
```

**What happens after you run this:**

```
1. CLI validates your agent locally (gitagent validate)
2. CLI forks the open-gitagent/registry repository to your account
3. CLI creates a metadata file in agents/ folder of the fork
4. CLI opens a Pull Request against the official registry repo
5. Registry CI runs automated validation checks
6. A maintainer reviews and approves the PR (usually within 24-48 hours)
7. Your agent appears live on registry.gitagent.sh
```

### Valid category values

```
developer-tools   data-engineering  devops         compliance
security          documentation     testing        research
productivity      finance           customer-support creative
education         other
```

### What your registry listing will show

Once approved, anyone can find and run your agent like this:

```bash
# What anyone in the world can run after your agent is registered:
npx @open-gitagent/gitagent@latest run \
  -r https://github.com/YOUR_USERNAME/security-audit-agent \
  -a claude
```

It appears on the registry at:
`https://registry.gitagent.sh/agents/YOUR_USERNAME/security-audit-agent`

---

## 11. Quick Reference Card

### One-liner run commands (after pushing to GitHub)

```bash
# ── SECURITY AUDIT AGENT ───────────────────────────────────────

# Claude Code (interactive)
npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/security-audit-agent -a claude

# OpenAI (one-shot)
OPENAI_API_KEY=sk-... npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/security-audit-agent -a openai -p "Audit this code: <paste>"

# Groq via GitClaw (interactive)
gitclaw --dir ./agents/security-audit-agent --model groq:llama-3.3-70b-versatile

# OpenClaw (one-shot)
ANTHROPIC_API_KEY=sk-ant-... npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/security-audit-agent -a openclaw -p "Audit this code: <paste>"


# ── INCIDENT COMMANDER AGENT ───────────────────────────────────

npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/incident-commander-agent -a claude
OPENAI_API_KEY=sk-... npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/incident-commander-agent -a openai -p "Here are my error logs: <paste>"
gitclaw --dir ./agents/incident-commander-agent --model groq:llama-3.3-70b-versatile
ANTHROPIC_API_KEY=sk-ant-... npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/incident-commander-agent -a openclaw -p "Write runbook for: <describe incident>"


# ── API ARCHITECT AGENT ────────────────────────────────────────

npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/api-architect-agent -a claude
OPENAI_API_KEY=sk-... npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/api-architect-agent -a openai -p "Design API for: <describe>"
gitclaw --dir ./agents/api-architect-agent --model groq:llama-3.3-70b-versatile
ANTHROPIC_API_KEY=sk-ant-... npx @open-gitagent/gitagent@latest run -r https://github.com/YOU/api-architect-agent -a openclaw -p "Review this API: <paste>"
```

### Environment variables summary

| Provider | Variable | Where to get it |
|----------|----------|----------------|
| Claude Code | (browser login) | `claude` → browser |
| OpenAI | `OPENAI_API_KEY` | platform.openai.com/api-keys |
| Groq | `GROQ_API_KEY` | console.groq.com (free tier) |
| OpenClaw / Anthropic | `ANTHROPIC_API_KEY` | console.anthropic.com |
| GitHub Models | `GITHUB_TOKEN` | github.com → Settings → Tokens |

### Full workflow from zero to registry

```bash
# 1. Install CLIs
npm install -g @open-gitagent/gitagent
gh auth login

# 2. Validate all 3 agents
gitagent validate -d ./agents/security-audit-agent
gitagent validate -d ./agents/incident-commander-agent
gitagent validate -d ./agents/api-architect-agent

# 3. Test locally (Claude)
gitagent run -d ./agents/security-audit-agent -a claude -p "Hello, what can you audit?"

# 4. Push all 3 to GitHub
cd agents/security-audit-agent && git init && git add . && git commit -m "feat: v1.0.0" && gh repo create security-audit-agent --public --source=. --push && git tag v1.0.0 && git push origin v1.0.0 && cd ..
cd incident-commander-agent && git init && git add . && git commit -m "feat: v1.0.0" && gh repo create incident-commander-agent --public --source=. --push && git tag v1.0.0 && git push origin v1.0.0 && cd ..
cd api-architect-agent && git init && git add . && git commit -m "feat: v1.0.0" && gh repo create api-architect-agent --public --source=. --push && git tag v1.0.0 && git push origin v1.0.0 && cd ..

# 5. Register on registry
gitagent registry -r https://github.com/YOU/security-audit-agent -c security -a claude-code,openai,system-prompt
gitagent registry -r https://github.com/YOU/incident-commander-agent -c devops -a claude-code,openai,system-prompt
gitagent registry -r https://github.com/YOU/api-architect-agent -c developer-tools -a claude-code,openai,system-prompt
```

---

## Troubleshooting

**`gitagent: command not found`**
```bash
npm install -g @open-gitagent/gitagent
# If still not found, check npm global bin is in PATH:
npm config get prefix  # should be in your PATH
```

**`claude: command not found`**
```bash
npm install -g @anthropic-ai/claude-code
claude  # opens browser auth
```

**`OPENAI_API_KEY not set`**
```bash
export OPENAI_API_KEY=sk-your-key
# Or add to ~/.bashrc / ~/.zshrc for persistence
```

**`gitclaw: command not found`**
```bash
npm install -g gitclaw
# Or use one-liner: bash <(curl -fsSL "https://raw.githubusercontent.com/open-gitagent/gitclaw/main/install.sh")
```

**Agent runs but gives generic answers**
Make sure `SOUL.md` is present and non-empty. This is the system prompt.
Run `gitagent info -d ./agent-folder` to see what the agent has loaded.

**Registry PR fails CI validation**
```bash
gitagent validate -d ./your-agent  # fix any errors shown
# Common issues: missing README.md, missing author in agent.yaml
```

---

> **GitAgent:** https://github.com/open-gitagent/gitagent
> **Registry:** https://registry.gitagent.sh
> **GitClaw:** https://github.com/open-gitagent/gitclaw
> **OpenClaw:** https://docs.openclaw.ai
> **Groq (free tier):** https://console.groq.com
