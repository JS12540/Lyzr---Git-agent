# GitAgent — Architecture & Directory Structure

> Deep dive into every file, directory, and system component that makes up a GitAgent-compliant repository.

---

## Table of Contents

1. [Full Directory Structure](#full-directory-structure)
2. [File-by-File Reference](#file-by-file-reference)
   - [agent.yaml — The Manifest](#agentyaml--the-manifest)
   - [SOUL.md — Agent Identity](#soulmd--agent-identity)
   - [RULES.md — Hard Constraints](#rulesmd--hard-constraints)
   - [DUTIES.md — Segregation of Duties](#dutiesmd--segregation-of-duties)
   - [AGENTS.md — Fallback Instructions](#agentsmd--fallback-instructions)
   - [skills/ — Capability Modules](#skills--capability-modules)
   - [tools/ — MCP Tool Definitions](#tools--mcp-tool-definitions)
   - [workflows/ — Playbooks](#workflows--playbooks)
   - [knowledge/ — Reference Documents](#knowledge--reference-documents)
   - [memory/ — Persistent State](#memory--persistent-state)
   - [hooks/ — Lifecycle Handlers](#hooks--lifecycle-handlers)
   - [compliance/ — Regulatory Config](#compliance--regulatory-config)
   - [agents/ — Sub-Agents](#agents--sub-agents)
   - [config/ — Environment Overrides](#config--environment-overrides)
   - [.gitagent/ — Runtime State](#gitagent--runtime-state)
3. [Architecture Diagrams](#architecture-diagrams)
4. [The Git-as-Supervision Layer](#the-git-as-supervision-layer)
5. [Memory System Deep Dive](#memory-system-deep-dive)
6. [Skills Framework Deep Dive](#skills-framework-deep-dive)

---

## Full Directory Structure

```
my-agent/
│
├── agent.yaml              ← REQUIRED: Central manifest
├── SOUL.md                 ← REQUIRED: Agent identity & personality
│
├── RULES.md                ← Hard behavioral constraints
├── DUTIES.md               ← Segregation of duties policy
├── AGENTS.md               ← Framework-agnostic fallback instructions
│
├── skills/                 ← Reusable capability modules
│   └── <skill-name>/
│       ├── SKILL.md        ← Frontmatter metadata + instructions
│       ├── scripts/        ← Executable helpers (.py / .sh / .js)
│       ├── references/     ← Supporting documentation
│       ├── assets/         ← Templates, schemas, examples
│       └── examples/       ← Calibration interactions
│
├── tools/                  ← MCP-compatible tool definitions
│   ├── <tool-name>.yaml    ← Tool schema
│   └── <tool-name>.py      ← Optional implementation
│
├── workflows/              ← Multi-step procedures
│   ├── <name>.yaml         ← Structured workflow (SkillsFlow)
│   └── <name>.md           ← Narrative workflow description
│
├── knowledge/              ← Reference documents for retrieval
│   ├── index.yaml          ← Retrieval hints & doc registry
│   └── *.md / *.pdf        ← Domain knowledge files
│
├── memory/                 ← Cross-session persistent state
│   ├── MEMORY.md           ← Current state (200-line max)
│   ├── memory.yaml         ← Memory configuration
│   ├── runtime/
│   │   ├── context.md      ← Current working context
│   │   ├── dailylog.md     ← Chronological activity log
│   │   └── key-decisions.md ← Important decisions log
│   └── archive/            ← Historical memory snapshots
│
├── hooks/                  ← Lifecycle event handlers
│   ├── hooks.yaml          ← Hook configuration
│   ├── bootstrap.md        ← Agent startup instructions
│   └── teardown.md         ← Agent shutdown cleanup
│
├── compliance/             ← Regulatory artifacts
│   └── regulatory-map.yaml ← Framework-to-rule mappings
│
├── agents/                 ← Sub-agent definitions
│   ├── <sub-agent>/        ← Full sub-agent (recursive structure)
│   └── <sub-agent>.md      ← Lightweight single-file sub-agent
│
├── config/                 ← Environment-specific overrides
│   ├── default.yaml        ← Default configuration
│   └── production.yaml     ← Production overrides
│
├── examples/               ← Calibration interactions
│   └── *.md
│
└── .gitagent/              ← Runtime state (gitignored)
    └── ...
```

---

## File-by-File Reference

### `agent.yaml` — The Manifest

The **single source of truth** for what the agent is and how it should run. Only `name`, `version`, and `description` are required.

```yaml
spec_version: "0.1.0"
name: compliance-analyst          # kebab-case identifier
version: 1.0.0                    # semantic versioning
description: Financial compliance analysis agent

model:
  preferred: claude-opus-4-6      # primary model
  fallback: claude-sonnet-4-6     # if primary unavailable
  temperature: 0.1                # determinism for compliance
  max_tokens: 4096

# Which skill directories are active
skills:
  - regulatory-analysis
  - document-review
  - risk-assessment

# Which tool files are active
tools:
  - compliance-checker
  - document-fetcher

runtime:
  max_turns: 50
  timeout: 300                    # seconds
  temperature: 0.1

# Inherit from a base agent
extends: https://github.com/org/base-analyst.git

# Dependencies on other agents
dependencies:
  - name: fact-checker
    source: https://github.com/org/fact-checker.git
    version: ^1.0.0
    mount: agents/fact-checker

# Full compliance config (see compliance section)
compliance:
  risk_tier: high                 # low | standard | high | critical
  frameworks:
    - finra
    - federal_reserve
    - sec
  supervision:
    designated_supervisor: compliance-team
    review_cadence: weekly
    human_in_the_loop: always
    kill_switch: true
    escalation_triggers:
      - confidence_below: 0.7
      - topic: litigation
  recordkeeping:
    audit_logging: true
    log_format: json
    retention_period: 7y
    immutable: true
  model_risk:
    inventory_id: MRM-2024-001
    validation_cadence: quarterly
    drift_detection: true
    ongoing_monitoring: true
  segregation_of_duties:
    roles:
      - id: analyst
        description: Creates analysis reports
        permissions: [create, submit]
      - id: reviewer
        description: Reviews and approves reports
        permissions: [review, approve, reject]
    conflicts:
      - [analyst, reviewer]       # same agent cannot do both
    assignments:
      compliance-analyst: [analyst]
      fact-checker: [reviewer]
    enforcement: strict
```

---

### `SOUL.md` — Agent Identity

Replaces scattered system prompts. The **single place** where the agent's personality, tone, and core values live. Portable across all frameworks.

```markdown
# Agent Identity

## Who You Are
You are a meticulous financial compliance analyst with deep expertise in
FINRA regulations, SEC requirements, and Federal Reserve guidelines.

## Communication Style
- Be precise and cite specific regulation numbers (e.g., FINRA Rule 3110)
- Use formal language appropriate for regulatory contexts
- Never hedge on compliance requirements — be definitive
- Acknowledge uncertainty explicitly when present

## Core Values
- Accuracy above all: a wrong compliance call has serious consequences
- Transparency: always explain your reasoning chain
- Conservatism: when in doubt, escalate to human review

## What You Know
You have deep familiarity with:
- FINRA Rules 3110, 4511, 2210, and Reg Notice 24-09
- Federal Reserve SR 11-7 (model risk management)
- SEC Regulation S-P (privacy of consumer financial information)
- CFPB Circular 2022-03 (AI explainability requirements)
```

---

### `RULES.md` — Hard Constraints

Absolute behavioral boundaries that **cannot be overridden** by user instructions or prompt injection. Evaluated by every framework adapter.

```markdown
# Behavioral Rules

## MUST ALWAYS
- Cite the specific regulation or rule number behind every recommendation
- Flag any ambiguity to human reviewers rather than making assumptions
- Log all decisions with timestamps and reasoning chains
- Request clarification when instructions conflict with regulations

## MUST NEVER
- Approve communications that contain promissory language
- Produce analysis without disclosing confidence level
- Access or process data outside designated classification level
- Bypass human review for high-risk decisions (risk_tier: high)
- Store PII outside designated, encrypted storage locations
```

---

### `DUTIES.md` — Segregation of Duties

Defines **who can do what** — and what conflicts exist. Enforces the four-eyes principle for regulated workflows.

```markdown
# Segregation of Duties Policy

## Roles

### analyst
- MAY: create reports, submit for review, query databases
- MAY NOT: approve own work, modify approved records

### reviewer
- MAY: review submissions, approve, reject, request changes
- MAY NOT: create the submissions they review

## Conflict Matrix
| Role A   | Role B   | Conflict? |
|----------|----------|-----------|
| analyst  | reviewer | YES — same entity cannot hold both |

## Handoff Requirements
- Credit decisions: require maker + checker approval
- Regulatory filings: require analyst + reviewer + supervisor
```

---

### `AGENTS.md` — Fallback Instructions

Framework-agnostic instructions that any runtime can read when the framework-specific export is not available. Acts as a universal fallback.

```markdown
# Agent Instructions (Framework-Agnostic)

This agent is a compliance analyst. When running this agent:

1. Load SOUL.md as the primary system prompt
2. Apply all rules from RULES.md as hard constraints
3. Respect segregation of duties defined in DUTIES.md
4. Available skills are listed in skills/ — load SKILL.md for each
5. Available tools are in tools/*.yaml — implement as function calls
6. Persist state to memory/runtime/context.md after each session
```

---

### `skills/` — Capability Modules

Reusable, composable capability packages. Each skill is a self-contained directory.

```
skills/
├── regulatory-analysis/
│   ├── SKILL.md          ← Frontmatter + instructions
│   ├── scripts/
│   │   └── parse_regs.py ← Helper scripts
│   ├── references/
│   │   └── finra_rules.md ← Supporting docs
│   └── examples/
│       └── sample_analysis.md ← Calibration examples
```

**`SKILL.md` format:**
```markdown
---
name: regulatory-analysis
version: 1.0.0
description: Analyze documents against FINRA and SEC requirements
author: compliance-team
tags: [compliance, finra, sec, analysis]
inputs:
  - document: string
  - frameworks: list
outputs:
  - findings: list
  - risk_score: float
  - recommendations: list
---

# Regulatory Analysis Skill

When analyzing a document:
1. Identify all regulatory touchpoints
2. Map each to applicable rule numbers
3. Score risk on scale 1-10
4. Provide specific remediation for each finding
```

**Progressive Disclosure (Three-Tier Loading):**
```
Tier 1 — Metadata only (always loaded):   name, version, description, tags
Tier 2 — Instructions (loaded on invoke): SKILL.md body content
Tier 3 — Full resources (loaded on use):  scripts/, references/, assets/
```
This prevents context window bloat — skills load progressively as needed.

---

### `tools/` — MCP Tool Definitions

MCP-compatible (Model Context Protocol) tool schemas. Any framework that supports MCP can use these directly.

```yaml
# tools/document-fetcher.yaml
name: document-fetcher
version: 1.0.0
description: Fetches regulatory documents from approved sources
schema:
  type: object
  properties:
    url:
      type: string
      description: Document URL (must be in approved_domains list)
    format:
      type: string
      enum: [pdf, html, txt]
  required: [url]
implementation: document-fetcher.py   # optional — points to local script
```

---

### `workflows/` — Playbooks

Structured multi-step procedures using **SkillsFlow** format.

```yaml
# workflows/compliance-review.yaml
name: compliance-review-flow
version: 1.0.0
description: Full compliance review pipeline
triggers:
  - document_submitted

steps:
  extract:
    skill: document-parser
    inputs:
      document: ${{ trigger.document }}

  analyze:
    skill: regulatory-analysis
    depends_on: [extract]
    inputs:
      content: ${{ steps.extract.outputs.text }}
      frameworks: [finra, sec]

  review:
    agent: fact-checker          # delegates to sub-agent
    depends_on: [analyze]
    prompt: |
      Verify each finding in the analysis report.
      Flag any finding with confidence below 0.8 for human review.
    inputs:
      findings: ${{ steps.analyze.outputs.findings }}

  report:
    skill: report-generator
    depends_on: [review]
    conditions:
      - ${{ steps.review.outputs.verified == true }}
    inputs:
      findings: ${{ steps.analyze.outputs.findings }}
      verification: ${{ steps.review.outputs.report }}
```

**Key SkillsFlow features:**
- `depends_on`: explicit step ordering
- `${{ }}`: template variable interpolation
- `conditions`: conditional execution gates
- `agent:`: delegate a step to a sub-agent

---

### `knowledge/` — Reference Documents

Domain knowledge the agent can consult during execution. Indexed for retrieval.

```yaml
# knowledge/index.yaml
documents:
  - id: finra-3110
    path: finra_rule_3110.md
    description: FINRA Rule 3110 — Supervision requirements
    tags: [finra, supervision, compliance]
    priority: high

  - id: sr-11-7
    path: federal_reserve_sr_11_7.md
    description: Federal Reserve SR 11-7 — Model Risk Management
    tags: [federal_reserve, model_risk]
    priority: high
```

---

### `memory/` — Persistent State

The most architecturally distinctive part of GitAgent. Long-term memory stored as **version-controlled Markdown** — not inside opaque vector databases.

```
memory/
├── MEMORY.md           ← Current state snapshot (max 200 lines)
├── memory.yaml         ← Memory configuration
├── runtime/
│   ├── context.md      ← What the agent is currently working on
│   ├── dailylog.md     ← Chronological activity log
│   └── key-decisions.md ← Log of important decisions
└── archive/
    └── 2026-03-01/     ← Archived snapshots by date
```

**Why this matters:**

| Traditional Agent Memory | GitAgent Memory |
|--------------------------|-----------------|
| Opaque vector database | Plain Markdown files |
| Hard to inspect | `cat memory/runtime/context.md` |
| No version history | Full `git log` history |
| Rollback impossible | `git revert` to any point |
| Not auditable | Complete audit trail |
| Siloed per-framework | Portable across frameworks |

**`memory/runtime/context.md` example:**
```markdown
# Current Context

## Active Task
Reviewing Q1 2026 marketing communications for FINRA compliance
Deadline: 2026-03-31

## Key Decisions Made
- Flagged 3 communications for promissory language (FINRA Rule 2210)
- Escalated 1 communication to supervisor (risk_tier: critical)

## Pending Actions
- [ ] Review remaining 12 communications
- [ ] Generate final report
```

---

### `hooks/` — Lifecycle Handlers

Scripts or instructions that run at specific agent lifecycle events.

```yaml
# hooks/hooks.yaml
hooks:
  bootstrap:
    - load: memory/runtime/context.md
    - validate: compliance/regulatory-map.yaml
    - log: "Agent started at {timestamp}"
  
  teardown:
    - persist: memory/runtime/context.md
    - archive: memory/runtime/dailylog.md
    - log: "Agent stopped, state persisted"
  
  on_error:
    - escalate: compliance-team
    - log_level: error
```

---

### `compliance/` — Regulatory Config

Houses regulatory artifacts and mapping files.

```yaml
# compliance/regulatory-map.yaml
frameworks:
  finra:
    rules:
      - id: "3110"
        title: "Supervision"
        applies_to: [all_communications, trade_reviews]
      - id: "4511"
        title: "Making and Preserving Books and Records"
        applies_to: [audit_logging, recordkeeping]
      - id: "2210"
        title: "Communications with the Public"
        applies_to: [marketing_review, social_media]
  
  federal_reserve:
    rules:
      - id: "SR-11-7"
        title: "Guidance on Model Risk Management"
        applies_to: [model_validation, ongoing_monitoring]
```

---

### `agents/` — Sub-Agents

Two patterns for defining sub-agents that can be orchestrated by a parent:

**Full sub-agent** (recursive structure):
```
agents/
└── fact-checker/       ← Has its own complete agent.yaml + SOUL.md
    ├── agent.yaml
    ├── SOUL.md
    └── skills/
```

**Lightweight sub-agent** (single file):
```markdown
---
name: fact-checker
model: claude-haiku-4-5-20251001
description: Verifies factual claims in analysis reports
---

You are a fact-checker. Given a compliance analysis report,
verify each factual claim against the cited regulatory text.
Flag any claim where the cited rule does not support the conclusion.
```

---

### `config/` — Environment Overrides

Environment-specific settings that override `agent.yaml` defaults.

```yaml
# config/production.yaml
model:
  temperature: 0.0      # maximum determinism in production
compliance:
  supervision:
    human_in_the_loop: always
  recordkeeping:
    immutable: true
runtime:
  max_turns: 25         # tighter limits in production
```

---

## Architecture Diagrams

### Complete File Relationship Map

```mermaid
graph LR
    subgraph CORE["Core (Required)"]
        AY[agent.yaml]
        SM[SOUL.md]
    end

    subgraph BEHAVIOR["Behavior Layer"]
        RM[RULES.md]
        DM[DUTIES.md]
        AM[AGENTS.md]
    end

    subgraph CAPABILITIES["Capability Layer"]
        SK[skills/]
        TL[tools/]
        WF[workflows/]
        KN[knowledge/]
    end

    subgraph STATE["State Layer"]
        MEM[memory/]
        HK[hooks/]
        CFG[config/]
    end

    subgraph COMPLIANCE["Compliance Layer"]
        CO[compliance/]
        AG[agents/]
    end

    AY -->|activates| SK
    AY -->|activates| TL
    AY -->|references| CO
    AY -->|extends| AG

    SM -->|loaded into| AM
    RM -->|enforced by| AM
    DM -->|enforced by| CO

    SK -->|used in| WF
    TL -->|used in| WF
    KN -->|consulted by| SK

    MEM -->|updated via| HK
    CFG -->|overrides| AY

    style CORE fill:#1e3a5f,color:#fff
    style BEHAVIOR fill:#2d5a27,color:#fff
    style CAPABILITIES fill:#5a3a1a,color:#fff
    style STATE fill:#3a1a5a,color:#fff
    style COMPLIANCE fill:#5a1a1a,color:#fff
```

---

### Agent Execution Flow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant CLI as gitagent CLI
    participant Repo as Git Repo (Agent)
    participant Adapter as Framework Adapter
    participant LLM as LLM (Claude/GPT/etc)

    Dev->>CLI: gitagent run --adapter claude
    CLI->>Repo: Read agent.yaml
    CLI->>Repo: Read SOUL.md
    CLI->>Repo: Read RULES.md
    CLI->>Repo: Read active skills/
    CLI->>Repo: Read tools/*.yaml
    CLI->>Repo: Read memory/runtime/context.md
    CLI->>Adapter: Build framework-specific config
    Adapter->>LLM: Send composed system prompt + tools
    LLM-->>Adapter: Response
    Adapter->>CLI: Return result
    CLI->>Repo: Update memory/runtime/context.md
    CLI->>Repo: Append to memory/runtime/dailylog.md
    CLI-->>Dev: Output
```

---

### Three-Layer Separation

```mermaid
graph TB
    subgraph L1["Layer 1: Identity (GitAgent)"]
        direction LR
        WHO[Who the agent IS\nSOUL.md]
        RULES[What it MUST/MUST NOT do\nRULES.md]
        SKILLS[What it CAN do\nskills/]
        TOOLS[What tools it uses\ntools/]
        MEM2[What it remembers\nmemory/]
    end

    subgraph L2["Layer 2: Definition (agent.yaml)"]
        direction LR
        MODEL[Model preferences]
        DEPS[Dependencies]
        COMP[Compliance config]
        ENV[Environment config]
    end

    subgraph L3["Layer 3: Execution (Framework)"]
        direction LR
        ORCH[Orchestration]
        STATE2[State machines]
        GRAPH[Graph wiring]
        API[API calls]
    end

    L1 --> L2
    L2 --> L3

    style L1 fill:#1e3a5f,color:#fff,stroke:#4a90d9
    style L2 fill:#2d5a27,color:#fff,stroke:#5cb85c
    style L3 fill:#5a3a1a,color:#fff,stroke:#e67e22
```

---

## The Git-as-Supervision Layer

This is GitAgent's most architecturally significant innovation. **Every change to agent behavior is a git commit.**

```mermaid
graph TD
    subgraph CHANGE["Agent State Change"]
        MC[Memory update\ncontext.md changed]
        SC[Skill acquired\nskills/ modified]
        PC[Personality drift\nSOUL.md modified]
        RC[Rule change\nRULES.md modified]
    end

    subgraph GIT["Git Workflow"]
        BR[New branch created\nagent/memory-update-2026-03-22]
        PR[Pull Request opened\n'Agent learned X, review behavior change']
        DIFF[Human reviews diff\ngit diff shows exact changes]
        APPROVE{Human approves?}
        MERGE[Merge to main]
        REJECT[git revert\nBehavior rolled back]
    end

    subgraph CI["CI/CD Checks"]
        VAL[gitagent validate]
        AUD[gitagent audit]
        COMP[Compliance check]
    end

    CHANGE --> BR
    BR --> PR
    PR --> DIFF
    PR --> CI
    CI --> VAL
    CI --> AUD
    CI --> COMP
    CI --> APPROVE
    DIFF --> APPROVE
    APPROVE -->|Yes| MERGE
    APPROVE -->|No| REJECT

    style CHANGE fill:#1e3a5f,color:#fff
    style GIT fill:#2d5a27,color:#fff
    style CI fill:#5a3a1a,color:#fff
    style APPROVE fill:#5a1a1a,color:#fff
```

**What this enables:**

| Git Operation | Agent Management Equivalent |
|---------------|----------------------------|
| `git log` | Full history of every behavior change |
| `git diff v1.0 v2.0` | Exactly what changed between agent versions |
| `git blame SOUL.md` | Who wrote each line of the agent's personality |
| `git revert abc123` | Roll back a broken prompt or drifted memory |
| `git branch staging` | Test new agent behavior in isolation |
| `git tag v1.1.0` | Pin production to a stable agent version |
| Pull Request | Human review of behavior changes before deploy |
| CI/CD check | Automated validation of agent spec on every push |

---

## Memory System Deep Dive

```mermaid
graph LR
    subgraph SESSION["Agent Session"]
        TASK[Task execution]
        DEC[Decision made]
        LEARN[New information]
    end

    subgraph MEMORY["memory/ directory"]
        CTX[runtime/context.md\nCurrent working state]
        LOG[runtime/dailylog.md\nActivity log]
        KD[runtime/key-decisions.md\nDecision log]
        MEM_IDX[MEMORY.md\n200-line snapshot]
    end

    subgraph ARCHIVE["memory/archive/"]
        SNAP[Daily snapshots\nby date]
    end

    subgraph GITLAYER["Git Layer"]
        COMMIT[git commit\n'Update agent memory']
        HISTORY[git log\nFull memory history]
        REVERT[git revert\nUndo memory state]
    end

    SESSION --> MEMORY
    MEMORY -->|"nightly"| ARCHIVE
    MEMORY --> GITLAYER
    GITLAYER -->|"next session"| MEMORY

    style SESSION fill:#1e3a5f,color:#fff
    style MEMORY fill:#2d5a27,color:#fff
    style ARCHIVE fill:#3a1a5a,color:#fff
    style GITLAYER fill:#5a3a1a,color:#fff
```

---

## Skills Framework Deep Dive

```mermaid
graph TB
    subgraph SKILL_STRUCT["Skill Directory Structure"]
        SKILL_MD[SKILL.md\nMetadata + Instructions]
        SCRIPTS[scripts/\nExecutable helpers]
        REFS[references/\nSupporting docs]
        ASSETS[assets/\nTemplates + schemas]
        EXAMPLES[examples/\nCalibration samples]
    end

    subgraph LOADING["Progressive Disclosure Loading"]
        T1["Tier 1 — Always\nname, version, tags"]
        T2["Tier 2 — On invoke\nInstructions body"]
        T3["Tier 3 — On use\nscripts/, refs/, assets/"]
    end

    subgraph LIFECYCLE["Skill Lifecycle"]
        SEARCH[gitagent skills search]
        INSTALL[gitagent skills install]
        LIST[gitagent skills list]
        INFO[gitagent skills info]
    end

    SKILL_MD --> T1
    SKILL_MD --> T2
    SCRIPTS --> T3
    REFS --> T3
    ASSETS --> T3

    SEARCH --> INSTALL
    INSTALL --> LIST
    LIST --> INFO

    style SKILL_STRUCT fill:#1e3a5f,color:#fff
    style LOADING fill:#2d5a27,color:#fff
    style LIFECYCLE fill:#5a3a1a,color:#fff
```

---

> **Sources:**
> - [GitAgent Specification v0.1.0](https://github.com/open-gitagent/gitagent/blob/main/spec/SPECIFICATION.md)
> - [GitHub — open-gitagent/gitagent](https://github.com/open-gitagent/gitagent)
> - [GitAgent Official Site](https://www.gitagent.sh/)
> - [MarkTechPost — Meet GitAgent](https://www.marktechpost.com/2026/03/22/meet-gitagent-the-docker-for-ai-agents-that-is-finally-solving-the-fragmentation-between-langchain-autogen-and-claude-code/)
