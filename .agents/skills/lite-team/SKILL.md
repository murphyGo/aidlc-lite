---
name: lite-team
description: Analyzes project signals and proposes a lightweight role-specialized agent team.
---

# Lite Team Skill

Generate a project-aware agent team. Team lead is required; specialists are proposed based on project signals and approved by the user before any files are written.

## Arguments

- `$ARGUMENTS` — (optional)
  - empty: full analysis + proposal flow
  - `status`: list existing `.agents/agents/` and stop
  - `add <role>`: skip analysis, propose adding a single role to an existing team
  - `reset`: archive `.agents/agents/` to `.agents/agents.bak/` and start fresh

## Objective

One round of analysis → one proposal → user approval → generated agents. **No stages.** The team operates on the same AIDLC artifacts (`BRIEF.md`, `PLAN.md`, `DECISIONS.md`) as `/lite-dev` — agents are role-specialized parallel workers, not a new workflow. The commit remains the only gate.

## Execution Steps

### Step 1: Pre-flight

1. **AIDLC artifacts must exist.** Read `BRIEF.md`, `PLAN.md`, `DECISIONS.md`. If any is missing, stop and tell the user to run `/lite-init` first.
2. **Existing team check.** If `.agents/agents/` exists with files:
   - `status` mode: list them with one-line `description` from each file's frontmatter and stop.
   - default mode: ask "Existing team found ({list}). Add to it (A), regenerate from scratch (R — moves current to `.agents/agents.bak/`), or cancel (C)?"
   - `reset` mode: do the move silently and proceed.
   - `add <role>` mode: skip Step 2 + Step 3, jump to Step 4 with a single-role proposal.
3. **Read project AGENTS.md** for tech stack and conventions.

### Step 2: Project analysis (read-only)

Sample, don't exhaustively read.

**Tech stack signals** (manifest files):
- `package.json` deps → `react`/`vue`/`svelte`/`next` (frontend), `express`/`fastify`/`koa` (backend), `jest`/`vitest`/`playwright` (testing), auth libs (`jsonwebtoken`, `passport`, `next-auth`)
- `pyproject.toml` / `requirements.txt` → `numpy`/`pandas`/`torch`/`tensorflow`/`scikit-learn` (data/ML), `fastapi`/`flask`/`django` (backend), `pytest` (testing)
- `go.mod`, `Cargo.toml`, `pom.xml` → standard ecosystem
- Top-level: `Dockerfile`, `docker-compose.yml`, `k8s/`, `terraform/`, `infra/`, `.github/workflows/`

**AIDLC signals**:
- `BRIEF.md` "Out of scope" — what's explicitly NOT being built (don't propose roles for excluded areas)
- `PLAN.md` length and item granularity (vague multi-area items → planner; many independent items → more devs)
- `DECISIONS.md` count (mature project = many entries; greenfield = few)

**Domain signals** in BRIEF or code:
- "auth", "login", "session", "PII", "encryption", "token" → security relevant
- "scrape", "research", "external API", "fetch", "summarize the web" → researcher useful
- "performance", "latency", "throughput", "p99" → performance-aware

### Step 3: Propose team composition

Always include exactly one **`team-lead`**. Then propose specialists from the catalog below. For each, give a one-line *why* tied to a detected signal. Suggest counts for parallel-friendly roles based on PLAN.md.

| Role | Default trigger | Default count |
|------|-----------------|---------------|
| `team-lead` | always | 1 |
| `planner` | PLAN.md items vague OR BRIEF gaps | 1 |
| `developer` | always | 1–3 (more if PLAN has many independent items) |
| `qa` | test framework configured OR BRIEF mentions reliability | 1 |
| `reviewer` | always | 1 |
| `doc-writer` | library/API project with sparse README | 1 |
| `security-reviewer` | auth/crypto/PII signals | 1 |
| `data-engineer` | numpy/pandas/torch/tf detected | 1 |
| `frontend` | React/Vue/Svelte/Next detected | 1–2 |
| `infra-operator` | Dockerfile/k8s/terraform detected | 1 |
| `researcher` | BRIEF references external research/scraping | 1 |

If the project clearly needs a role outside this catalog, propose a new one with a name and reasoning. Examples that often emerge: `prompt-engineer` (LLM apps), `migration-runner` (DB-heavy), `ux-writer` (content-heavy UI).

**Parallelism heuristic for `developer`:**

| PLAN.md state | Default dev count |
|---------------|-------------------|
| < 5 unchecked items | 1 |
| 5–15 unchecked, mostly independent | 2 |
| 15+ unchecked, multiple modules | 3 |

Present the proposal as:

```
Proposed team for <project name>:

Required:
  • team-lead × 1 — orchestrator, AIDLC-aware

Recommended (signal → role):
  • developer × 2 — PLAN.md has 9 independent tasks
  • qa × 1 — pytest configured in pyproject.toml
  • reviewer × 1 — default
  • frontend × 1 — package.json has 'next' and 'react'

Optional (you can add):
  • doc-writer, planner, researcher, security-reviewer

Approve as-is, edit (e.g., "drop qa, +1 developer, add researcher"), or cancel?
```

### Step 4: Approval loop (one round, hard cap)

Wait for user response. Accept:
- "approve" / "yes" / "ok" / "go" → proceed to Step 5
- Edit instructions ("drop X, add Y, dev count = 3") → adjust once and re-show
- "cancel" / "stop" → stop without writing anything

If the user keeps re-editing past two rounds, ask "Lock this in?" and proceed on approval. **Do not loop indefinitely.**

### Step 5: Generate agent files

For each approved role and count, write `.agents/agents/<name>.md`. Multi-instance roles get `<name>-1`, `<name>-2`, ... — each instance gets a slightly different focus hint in its description if useful (e.g., `developer-1: backend tasks`, `developer-2: frontend tasks`).

Read the base template at `.agents/skills/lite-team/templates/agent-base.md` for the structural skeleton (frontmatter + AIDLC awareness block + out-of-bounds block). Fill in role-specific content from this matrix:

| Role | Tools | Mission | Out-of-bounds extras |
|------|-------|---------|-----------------------|
| `team-lead` | (omit `tools` to inherit all) | Read PLAN; pick the next unchecked item OR propose a new one if PLAN is exhausted; do directly OR delegate via the Agent tool to specialists; integrate results; mark PLAN; report to user. | Never modify `BRIEF.md` without explicit user instruction. Never run multiple destructive Bash commands without surfacing them first. |
| `planner` | `Read, Edit, Grep, Glob` | Refine PLAN.md granularity, surface BRIEF gaps, propose DECISIONS.md entries for non-obvious choices. | Don't write code. Don't reorder PLAN without team-lead approval. Edits limited to BRIEF/PLAN/DECISIONS. |
| `developer` | `Read, Write, Edit, Bash, Grep, Glob` | Implement one PLAN item end-to-end. Modify files in place. Run tests. Report. | Don't add features outside PLAN. No `_v2`/`_new`/`_modified` sibling files. Don't commit. |
| `qa` | `Read, Bash, Grep, Glob` | Write or run tests, find edge cases, report failures with file:line. | Don't modify production code — escalate fixes back to a developer. |
| `reviewer` | `Read, Grep, Glob, Bash` | Lightweight pre-commit check on staged or recent changes. Defers to `/code-review` for deep passes. | Read-only — never edit. |
| `doc-writer` | `Read, Write, Edit, Grep, Glob` | README, docstrings, API docs aligned with BRIEF.md. | Don't change code semantics. Don't restructure existing prose without need. |
| `security-reviewer` | `Read, Grep, Glob, Bash` | Scan for OWASP Top 10, secret leakage, auth bypass, unsafe deserialization. | Read-only. Escalate findings to user, don't auto-fix. |
| `data-engineer` | `Read, Write, Edit, Bash, Grep, Glob` | Data pipelines, model training, notebook work, schema design. | Don't touch unrelated app code. |
| `frontend` | `Read, Write, Edit, Bash, Grep, Glob` | UI components, styling, client-side state, accessibility. | Don't touch backend/API code without delegation. |
| `infra-operator` | `Read, Write, Edit, Bash, Grep, Glob` | Dockerfile, CI/CD, deploy scripts, infra-as-code. | Don't deploy or push to remote infra without user approval. |
| `researcher` | `Read, WebFetch, WebSearch, Grep, Glob` | External research; produce a markdown brief with citations. | Read-only inside the repo. Don't edit code. |

**Naming conventions:**
- File name = agent name = role (e.g., `team-lead.md`, `developer-1.md`).
- `description` field should be one sentence answering "when should I invoke this agent?" — that's what Codex uses to route.

### Step 6: Generate TEAM.md at project root

Read `.agents/skills/lite-team/templates/TEAM.md`, fill in the approved roster as a table, and write to project root `TEAM.md`. This is the team manifest the team-lead and user reference.

### Step 7: Update project AGENTS.md

Append (don't replace) a `## Team` section to project `AGENTS.md`:

```markdown
## Team

This project has an agent team. See `TEAM.md` for the roster.

- Default entry point for non-trivial work: invoke the `team-lead` agent (Agent tool with `subagent_type=team-lead`).
- Team lead reads `PLAN.md` and either does the work directly or delegates to specialists.
- Specialists report back; the lead integrates. The user remains the only commit gate.
```

If a `## Team` section already exists, refresh the roster line and leave the rest.

### Step 8: Final report

```
✓ Generated <N> agents in .agents/agents/
✓ TEAM.md written at project root
✓ AGENTS.md updated with Team section

Roster:
  - team-lead
  - developer × 2
  - qa
  - reviewer
  - frontend

Invoke the team lead by asking Codex to "have team-lead pick up the next PLAN item",
or invoke directly with the Agent tool: subagent_type=team-lead.
```

**Do not** invoke the team-lead immediately. The user runs the team when they want it.

## Guidelines

- **One proposal, one approval.** Don't loop. Don't over-customize.
- **Team lead is the only required role.** Everything else is signal-driven.
- **Never invent new workflows.** Agents read BRIEF/PLAN/DECISIONS, just like `/lite-dev`. Those artifacts stay authoritative.
- **The commit is still the only gate.** No agent commits. The user reviews and commits.
- **Don't generate empty roles.** If no signal triggers a role, don't add it just for symmetry.
- **Resist agent inflation.** A 4–6 agent team is healthy. A 12-agent team means you're rebuilding aidlc-starter — stop.
- **Delegation, not stages.** The team lead delegates as needed; it does not run a fixed sequence of agents per task.

## Example invocations

- `/lite-team` — full analysis and proposal
- `/lite-team status` — list existing agents
- `/lite-team add data-engineer` — propose adding one role to an existing team
- `/lite-team reset` — archive existing agents and start fresh
