# How to Push to GitHub & Register on the GitAgent Registry

> Complete guide: GitHub setup → registry submission → PR review → Discord community.
> Written for the 3 agents: `security-audit-agent`, `incident-commander-agent`, `api-architect-agent`.

---

## Table of Contents

1. [Join the Community First](#1-join-the-community-first)
2. [What You Need to Create on GitHub](#2-what-you-need-to-create-on-github)
3. [Prerequisites — Install Everything](#3-prerequisites--install-everything)
4. [Push Agent 1 — security-audit-agent](#4-push-agent-1--security-audit-agent)
5. [Push Agent 2 — incident-commander-agent](#5-push-agent-2--incident-commander-agent)
6. [Push Agent 3 — api-architect-agent](#6-push-agent-3--api-architect-agent)
7. [How the GitAgent Registry Works](#7-how-the-gitagent-registry-works)
8. [Create Your Registry Entry (metadata.json)](#8-create-your-registry-entry-metadatajson)
9. [Fork the Registry and Open a Pull Request](#9-fork-the-registry-and-open-a-pull-request)
10. [What Happens After You Submit the PR](#10-what-happens-after-you-submit-the-pr)
11. [Full Command Cheatsheet](#11-full-command-cheatsheet)

---

## 1. Join the Community First

Before anything else, join the GitAgent community. This is where maintainers
help with registry reviews, people share agents, and you can ask questions.

### Discord Server

```
https://discord.gg/hVZV8Xyjdc
```

Click that link → it opens the official GitAgent Discord.
Once inside:
- Say hi in `#introductions` or `#general`
- Check `#registry` or `#agents` for submission tips
- Tag a maintainer if your PR is waiting for review

### GitHub — Star and Watch the repos

```
https://github.com/open-gitagent/gitagent     ← main framework (2.7k stars)
https://github.com/open-gitagent/registry     ← where your PR goes
https://github.com/open-gitagent/gitclaw      ← runtime (203 stars)
```

Click **Watch → All Activity** on the registry repo so you get notified
when your PR gets a comment or merge.

---

## 2. What You Need to Create on GitHub

### You need 3 separate GitHub repositories — one per agent

Each agent is its own repo. The repo IS the agent.

| Repository name | What goes in it |
|-----------------|-----------------|
| `security-audit-agent` | All files from `agents/security-audit-agent/` |
| `incident-commander-agent` | All files from `agents/incident-commander-agent/` |
| `api-architect-agent` | All files from `agents/api-architect-agent/` |

Each repo must be:
- **Public** (registry CI must be able to clone it)
- **Tagged** with `v1.0.0`
- Contains: `agent.yaml`, `SOUL.md`, `RULES.md`, `README.md`, `skills/*/SKILL.md`

### You also need to fork 1 registry repo

```
https://github.com/open-gitagent/registry
```

You fork this once, add your 3 agents to it, and open 3 PRs (or 1 PR with all 3).

---

## 3. Prerequisites — Install Everything

Run all of these once before starting:

```bash
# Check Node (need v18+)
node --version

# Install gitagent CLI
npm install -g @open-gitagent/gitagent
gitagent --version

# Install GitHub CLI
# Download from: https://cli.github.com
# Then authenticate:
gh auth login
# Choose: GitHub.com → HTTPS → Login with a web browser
# Follow the URL it gives you → paste the one-time code

# Confirm login worked
gh auth status
# Should show: Logged in to github.com as YOUR_USERNAME
```

---

## 4. Push Agent 1 — security-audit-agent

Run these commands **exactly in order**:

```bash
# 1. Go into the agent folder
cd C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent\agents\security-audit-agent

# 2. Initialise a git repo
git init

# 3. Stage all files
git add .

# 4. First commit
git commit -m "feat: initial release v1.0.0 — security-audit-agent"

# 5. Create the public GitHub repo AND push in one command
#    Replace jay-shah with your actual GitHub username
gh repo create security-audit-agent \
  --public \
  --description "AI security engineer — OWASP Top 10, secrets detection, CVE scanning" \
  --source=. \
  --push

# 6. Create a version tag (required for registry)
git tag v1.0.0
git push origin v1.0.0

# 7. Confirm it worked — open in browser
gh browse
```

Your agent is now live at:
```
https://github.com/YOUR_USERNAME/security-audit-agent
```

---

## 5. Push Agent 2 — incident-commander-agent

```bash
# 1. Go into the folder
cd C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent\agents\incident-commander-agent

# 2. Init + commit + push
git init
git add .
git commit -m "feat: initial release v1.0.0 — incident-commander-agent"

gh repo create incident-commander-agent \
  --public \
  --description "AI SRE for production incidents — log analysis, runbooks, post-mortems" \
  --source=. \
  --push

git tag v1.0.0
git push origin v1.0.0

gh browse
```

Live at:
```
https://github.com/YOUR_USERNAME/incident-commander-agent
```

---

## 6. Push Agent 3 — api-architect-agent

```bash
# 1. Go into the folder
cd C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent\agents\api-architect-agent

# 2. Init + commit + push
git init
git add .
git commit -m "feat: initial release v1.0.0 — api-architect-agent"

gh repo create api-architect-agent \
  --public \
  --description "AI API architect — OpenAPI 3.1 specs, endpoint review, developer docs" \
  --source=. \
  --push

git tag v1.0.0
git push origin v1.0.0

gh browse
```

Live at:
```
https://github.com/YOUR_USERNAME/api-architect-agent
```

---

## 7. How the GitAgent Registry Works

The registry lives at `https://registry.gitagent.sh`.
It is backed by the GitHub repo `https://github.com/open-gitagent/registry`.

```
open-gitagent/registry/
└── agents/
    ├── shreyas-lyzr__architect/
    │   ├── metadata.json      ← points to the GitHub repo
    │   └── README.md
    ├── AJAmit17__chrome-extension-builder/
    │   ├── metadata.json
    │   └── README.md
    └── YOUR_USERNAME__security-audit-agent/   ← you add this
        ├── metadata.json
        └── README.md
```

**Submission flow:**

```
You fork open-gitagent/registry
      ↓
You add:  agents/YOUR_USERNAME__security-audit-agent/metadata.json
          agents/YOUR_USERNAME__security-audit-agent/README.md
      ↓
You open a Pull Request to open-gitagent/registry
      ↓
Registry CI automatically:
  • validates metadata.json fields
  • clones your GitHub repo (must be public!)
  • confirms it is a valid GitAgent implementation
      ↓
Maintainer reviews → approves → merges
      ↓
index.json regenerated → your agent appears on registry.gitagent.sh
```

---

## 8. Create Your Registry Entry (metadata.json)

Each agent needs a `metadata.json` file in the registry.
The folder name format is: `YOUR_GITHUB_USERNAME__agent-name`

### security-audit-agent metadata

Create file: `agents/jay-shah__security-audit-agent/metadata.json`
```json
{
  "name": "security-audit-agent",
  "author": "jay-shah",
  "repository": "https://github.com/jay-shah/security-audit-agent",
  "version": "1.0.0",
  "description": "AI security engineer that audits code for OWASP Top 10 vulnerabilities, detects hardcoded secrets, and scans dependency files for known CVEs.",
  "category": "security",
  "tags": ["security", "owasp", "vulnerability-scanning", "secrets-detection", "cve"],
  "license": "MIT",
  "adapters": ["claude", "openai", "system-prompt"],
  "spec_version": "0.1.0",
  "skills": ["owasp-scanner", "secrets-detector", "dep-auditor"]
}
```

### incident-commander-agent metadata

Create file: `agents/jay-shah__incident-commander-agent/metadata.json`
```json
{
  "name": "incident-commander-agent",
  "author": "jay-shah",
  "repository": "https://github.com/jay-shah/incident-commander-agent",
  "version": "1.0.0",
  "description": "AI SRE that analyzes production logs to find root causes, writes runbooks for common failure modes, and generates blameless post-mortems.",
  "category": "devops",
  "tags": ["devops", "sre", "incident-response", "observability", "runbooks"],
  "license": "MIT",
  "adapters": ["claude", "openai", "system-prompt"],
  "spec_version": "0.1.0",
  "skills": ["log-analyzer", "runbook-writer", "postmortem-generator"]
}
```

### api-architect-agent metadata

Create file: `agents/jay-shah__api-architect-agent/metadata.json`
```json
{
  "name": "api-architect-agent",
  "author": "jay-shah",
  "repository": "https://github.com/jay-shah/api-architect-agent",
  "version": "1.0.0",
  "description": "AI API architect that designs production-grade REST APIs, generates complete OpenAPI 3.1 specifications, and reviews existing APIs for anti-patterns and breaking changes.",
  "category": "developer-tools",
  "tags": ["api-design", "openapi", "rest", "documentation", "developer-tools"],
  "license": "MIT",
  "adapters": ["claude", "openai", "system-prompt"],
  "spec_version": "0.1.0",
  "skills": ["openapi-designer", "endpoint-reviewer", "api-doc-writer"]
}
```

> **Replace `jay-shah` with your actual GitHub username everywhere above.**

---

## 9. Fork the Registry and Open a Pull Request

Do this **after** all 3 agent repos are live on GitHub.

### Step 1 — Fork the registry repo

```bash
# Go to a working folder (not inside any agent folder)
cd C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent

# Fork and clone the registry repo to your machine
gh repo fork open-gitagent/registry --clone
# This creates: open-gitagent/registry forked to YOUR_USERNAME/registry
# And clones it locally into a folder called "registry"

cd registry
```

### Step 2 — Create your agent entries

```bash
# Create folders for each agent
# Replace jay-shah with YOUR actual GitHub username

mkdir -p agents/jay-shah__security-audit-agent
mkdir -p agents/jay-shah__incident-commander-agent
mkdir -p agents/jay-shah__api-architect-agent
```

### Step 3 — Add metadata.json for each agent

Copy the 3 metadata.json contents from Section 8 above into each folder.

For example (using PowerShell/CMD):
```bash
# Create the metadata.json files
# Copy-paste the JSON from Section 8 into each file

# You can also use any text editor — create the file manually
```

### Step 4 — Add README.md for each agent

Copy the `README.md` from each agent folder into the registry folder:

```bash
# From inside the registry folder:
cp C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent\agents\security-audit-agent\README.md \
   agents/jay-shah__security-audit-agent/README.md

cp C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent\agents\incident-commander-agent\README.md \
   agents/jay-shah__incident-commander-agent/README.md

cp C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent\agents\api-architect-agent\README.md \
   agents/jay-shah__api-architect-agent/README.md
```

### Step 5 — Commit and push to your fork

```bash
# Inside the registry folder
git add .
git commit -m "feat: add security-audit-agent, incident-commander-agent, api-architect-agent by jay-shah"
git push origin main
```

### Step 6 — Open the Pull Request

```bash
gh pr create \
  --repo open-gitagent/registry \
  --title "feat: add 3 agents by jay-shah — security, incident-commander, api-architect" \
  --body "$(cat <<'EOF'
## New Agents

Adding 3 production-ready agents:

### 1. security-audit-agent
- Category: security
- Skills: owasp-scanner, secrets-detector, dep-auditor
- Repo: https://github.com/jay-shah/security-audit-agent

### 2. incident-commander-agent
- Category: devops
- Skills: log-analyzer, runbook-writer, postmortem-generator
- Repo: https://github.com/jay-shah/incident-commander-agent

### 3. api-architect-agent
- Category: developer-tools
- Skills: openapi-designer, endpoint-reviewer, api-doc-writer
- Repo: https://github.com/jay-shah/api-architect-agent

## Checklist
- [x] All repos are public
- [x] agent.yaml present in each repo
- [x] SOUL.md present in each repo
- [x] README.md present in each repo
- [x] skills/*/SKILL.md present in each repo
- [x] v1.0.0 tag pushed to each repo
- [x] gitagent validate passes locally
- [x] metadata.json added for each agent
- [x] spec_version: 0.1.0
EOF
)"
```

This opens the PR at:
```
https://github.com/open-gitagent/registry/pulls
```

---

## 10. What Happens After You Submit the PR

```
Day 0 — You open the PR
  ↓
  Registry CI starts automatically:
    ✅ validates metadata.json schema
    ✅ checks repository field is reachable (clones your GitHub repos)
    ✅ checks agent.yaml spec_version matches
    ✅ checks SOUL.md and README.md exist
  ↓
  If CI fails → fix the errors it reports, push to your fork, PR auto-updates
  ↓
Day 1-2 — A registry maintainer reviews the PR
  • They may leave comments asking for changes
  • Reply in the PR thread (or on Discord if urgent)
  ↓
Maintainer merges → CI regenerates index.json
  ↓
Your agents appear live on https://registry.gitagent.sh
```

### If CI fails — common fixes

| CI Error | Fix |
|----------|-----|
| `repository not accessible` | Make sure all 3 repos are **public** on GitHub |
| `missing required field in metadata.json` | Check all fields in Section 8 are present |
| `spec_version mismatch` | Set `"spec_version": "0.1.0"` in metadata.json |
| `agent.yaml not found` | Confirm the file is at the root of the repo, not in a subfolder |
| `SOUL.md missing` | Make sure SOUL.md is committed and pushed |

### While waiting — post in Discord

```
Discord: https://discord.gg/hVZV8Xyjdc
```

Post something like:
> "Just submitted a PR to add 3 agents — security-audit, incident-commander, and api-architect.
> PR: https://github.com/open-gitagent/registry/pulls/YOUR_PR_NUMBER
> Would appreciate a review!"

Maintainers are active in Discord and this speeds up the review.

---

## 11. Full Command Cheatsheet

Everything from zero to registered, in order:

```bash
# ── INSTALL CLIs ────────────────────────────────────────────────
npm install -g @open-gitagent/gitagent
gh auth login

# ── VALIDATE LOCALLY FIRST ──────────────────────────────────────
cd C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent
gitagent validate -d ./agents/security-audit-agent
gitagent validate -d ./agents/incident-commander-agent
gitagent validate -d ./agents/api-architect-agent

# ── PUSH AGENT 1 ────────────────────────────────────────────────
cd agents/security-audit-agent
git init && git add . && git commit -m "feat: initial release v1.0.0"
gh repo create security-audit-agent --public --description "AI security engineer — OWASP Top 10, secrets detection, CVE scanning" --source=. --push
git tag v1.0.0 && git push origin v1.0.0

# ── PUSH AGENT 2 ────────────────────────────────────────────────
cd ../incident-commander-agent
git init && git add . && git commit -m "feat: initial release v1.0.0"
gh repo create incident-commander-agent --public --description "AI SRE for production incidents — log analysis, runbooks, post-mortems" --source=. --push
git tag v1.0.0 && git push origin v1.0.0

# ── PUSH AGENT 3 ────────────────────────────────────────────────
cd ../api-architect-agent
git init && git add . && git commit -m "feat: initial release v1.0.0"
gh repo create api-architect-agent --public --description "AI API architect — OpenAPI 3.1 specs, endpoint review, developer docs" --source=. --push
git tag v1.0.0 && git push origin v1.0.0

# ── FORK REGISTRY & ADD ENTRIES ─────────────────────────────────
cd C:\Users\jay.shah\Desktop\Modulr_Projects\Learning\git_agent
gh repo fork open-gitagent/registry --clone
cd registry
mkdir -p agents/jay-shah__security-audit-agent
mkdir -p agents/jay-shah__incident-commander-agent
mkdir -p agents/jay-shah__api-architect-agent
# ↑ then create metadata.json in each folder (see Section 8)
# ↑ then copy README.md from each agent into each folder

git add .
git commit -m "feat: add 3 agents by jay-shah"
git push origin main

# ── OPEN THE PR ─────────────────────────────────────────────────
gh pr create --repo open-gitagent/registry \
  --title "feat: add security-audit, incident-commander, api-architect agents by jay-shah" \
  --body "Adding 3 production-ready agents. All repos public, tagged v1.0.0, spec v0.1.0."

# ── JOIN DISCORD ─────────────────────────────────────────────────
# https://discord.gg/hVZV8Xyjdc
# Post your PR link there for faster review
```

---

## All Important Links

| Resource | URL |
|----------|-----|
| **Registry website** | https://registry.gitagent.sh |
| **Registry GitHub repo** | https://github.com/open-gitagent/registry |
| **GitAgent main repo** | https://github.com/open-gitagent/gitagent |
| **GitAgent org** | https://github.com/open-gitagent |
| **GitClaw runtime** | https://github.com/open-gitagent/gitclaw |
| **Spec documentation** | https://github.com/open-gitagent/gitagent/blob/main/spec/SPECIFICATION.md |
| **Discord community** | https://discord.gg/hVZV8Xyjdc |
| **GitAgent website** | https://www.gitagent.sh |
| **Groq free API keys** | https://console.groq.com |
| **Anthropic API keys** | https://console.anthropic.com |
| **OpenAI API keys** | https://platform.openai.com/api-keys |
| **GitHub CLI install** | https://cli.github.com |
