# aidlc-lite (template repo)

This is the **aidlc-lite template repo itself**, not a project using it. After cloning, the user runs `/lite-init` and this file is replaced with a project-specific CLAUDE.md.

## Purpose

Lightweight personal AIDLC starter. Three files (`BRIEF.md`, `PLAN.md`, `DECISIONS.md`) and four skills (`/lite-init`, `/lite-dev`, `/code-review`, `/lite-team`).

## Skills

| Skill | Purpose |
|-------|---------|
| `/lite-init` | Idea capture entry point — auto-detects greenfield / brownfield / refinement, generates or updates BRIEF, PLAN, DECISIONS, project CLAUDE.md |
| `/lite-dev` | Picks next unchecked PLAN.md item, implements, checks it off |
| `/code-review` | Deep review of pending changes — language/framework patterns + 7 conditional protocols (concurrency, security, etc.) |
| `/lite-team` | Optional. Analyzes the project and proposes an agent team (team-lead + signal-driven specialists). User approves; skill writes `.claude/agents/*.md` + `TEAM.md` |

## Files

| File | Role |
|------|------|
| `README.md` | What aidlc-lite is, quick start, design principles |
| `.claude/skills/lite-init/SKILL.md` | Init skill definition |
| `.claude/skills/lite-init/templates/BRIEF.md` | Template for the project's one-page brief |
| `.claude/skills/lite-init/templates/PLAN.md` | Template for the project's flat checklist |
| `.claude/skills/lite-init/templates/DECISIONS.md` | Template for the project's ADR log |
| `.claude/skills/lite-dev/SKILL.md` | Dev driver skill definition |
| `.claude/skills/code-review/SKILL.md` | Code review skill (trimmed from aidlc-starter) |
| `.claude/skills/code-review/protocols/` | 7 deep-analysis protocols + INDEX (loaded on demand) |
| `.claude/skills/lite-team/SKILL.md` | Meta-skill: proposes and generates an agent team based on project signals |
| `.claude/skills/lite-team/templates/agent-base.md` | Skeleton for each generated agent (frontmatter + AIDLC awareness + out-of-bounds) |
| `.claude/skills/lite-team/templates/TEAM.md` | Template for the project's team manifest (written to project root on team gen) |

## When developing aidlc-lite itself

- Skills must stay short. `/lite-init` ≤ 220 lines, `/lite-dev` ≤ 150 lines, `/lite-team` ≤ 250 lines. If they grow, you're rebuilding full AIDLC.
- `/code-review` is the exception — it carries language/framework tables and protocols. Keep SKILL.md ≤ 300 lines; protocols are conditionally loaded so they don't burn context.
- Templates must stay prose-friendly. No FR-001/NFR-001 numbering schemes. No fill-in-the-blank forms.
- Resist adding stages. The whole value prop is "no stages."
- Resist adding `audit.md`, `aidlc-state.md`, refinement-log, cross-check. Git and checklists replace all of these.
- `/code-review` must NEVER become a mandatory gate inside `/lite-dev`. It is a suggestion only — the commit is the gate.
- `/lite-team` must NOT introduce hand-off documents, stage gates between agents, or any state outside `BRIEF.md` / `PLAN.md` / `DECISIONS.md` / `TEAM.md`. Agents are `/lite-dev` × N with role specialization, not a new workflow. Team lead is the only required role.
- If a feature is "useful for handoff to others" — it belongs in aidlc-starter, not here.

## Promotion path

Out of scope for now. If a user's project grows past ~3k LOC and they want full AIDLC, they manually copy `BRIEF.md` content into an aidlc-starter `IDEA.md` and run `/init-project` there. We do not maintain a migration tool.
